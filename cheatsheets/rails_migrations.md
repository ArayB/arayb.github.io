# Rails Migrations

#### Column Types

- :binary
- :boolean
- :date
- :datetime
- :decimal
- :float
- :integer
- :bigint
- :primary_key
- :references
- :string
- :text
- :time
- :timestamp

##### If using PostgreSQL

- :hstore
- :json
- :jsonb
- :array
- :cidr_address
- :ip_address
- :mac_address

Which will be stored as strings in a non PostgreSQL database.

[There are more PostgreSQL specific columns available in Rails 5](https://github.com/rails/rails/blob/master/activerecord/lib/active_record/connection_adapters/postgresql_adapter.rb#L69)



