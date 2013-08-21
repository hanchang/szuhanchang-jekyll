---
layout: post
title:  "Using UUID as Primary Key on Postgres"
date:   2013-08-16
tags: ranktracker.io
---

I finally bit the bullet and switched from using autoincrementing primary keys on PostgreSQL to UUIDs. The reasons are manyfold, but the driving factor was for security; since the ids are shown in the application URLs, I wanted to make them virtually impossible to guess. Additionally, it gives a lot of room for expansion in the future, as I can confidently shard without worrying about keeping IDs unique and in sync, especially since workers will be retrieving ranking data and linking it back to the keyword tuple via these IDs.

However, I didn't want to depend on a Postgres extension to do this. Every other tutorial online for using UUIDs as PKs in Postgres suggests using the 'uuid-ossp' extension and the `uuid_generate_v4()` method for generating the UUIDs, but I wasn't 100% confident I'd have access to that extension on the various SaaS providers so I figured I'd generate it myself in Ruby code via `SecureRandom.uuid()` in a `before_create` model callback method. That said, Heroku Postgres does offer support for the `uuid-ossp` extension (along with 16 others like `hstore`) which is great since that's what I plan on using.

I do realize that this could be a performance issue, so if I see substantial slowdowns in the app via something like New Relic I'll go back and benchmark the difference between Ruby UUID generation vs Postgres UUID generation.

Without further ado, here is how I got it working:

    # In your migration, for each model that uses UUIDs as PK:

    create_table :table_name, id: false do |t|
      t.primary_key :id, :uuid, default: nil
      t.string :something
      # ... Add your model logic here!
      t.timestamps
    end

    # In app/models/concerns/uuid.rb
    module UUID
      extend ActiveSupport::Concern

      included do
        before_create :generate_uuid
      end

      private
      def generate_uuid
        self.id = SecureRandom.uuid
      end
    end

    # In your model:

    class ModelName < ActiveRecord::Base
      include UUID
      # Rest of your model definition
    end

I went ahead and made the UUID logic a mixin in order to keep the code DRY since I had multiple model classes that would take advantage of this functionality.

For full disclosure, I learned this technique from this excellent commit into Rails core - [https://github.com/rails/rails/pull/10404/files](https://github.com/rails/rails/pull/10404/files). There's nothing like using the source!
