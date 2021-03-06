# SubstrateClient

This is a library of interfaces for communicating with Substrate nodes. It provides application developers the ability to query a node and interact with the Substrate chains using Ruby.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'substrate_client'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install substrate_client

## Usage

### Supported rpc methods

#### rpc method list

```ruby
require "substrate_client"

client = SubstrateClient.new("wss://kusama-rpc.polkadot.io/")
client.method_list do |methods|
  p methods
end
```
The rpc methods can be dynamically called by its name, so the methods returned by this method can all be used.



#### rpc methods

But, in order to show the parameters more clearly, some important or frequently used methods are hard-coded:

- chain_get_finalised_head(&callback)

  Get hash of the last finalized block in the canon chain

  ```ruby
  client.chain_get_finalised_head do |head|
    p head 
  end
  ```

  

- chain_get_head(&callback)

  Retrieves the header

  

- chain_get_header(block_hash = nil, &callback)

  Retrieves the header for a specific block

  

- chain_get_block(block_hash = nil, &callback)

  Get header and body of a relay chain block

  

- chain_get_block_hash(block_id, &callback)

  Get the block hash for a specific block

  

- chain_get_runtime_version(block_hash = nil, &callback)

  Get the runtime version for a specific block

  

- state_get_metadata(block_hash = nil, &callback)

  Returns the runtime metadata by block

  

- state_get_storage(storage_key, block_hash = nil, &callback)

  Retrieves the storage for a key

  

- system_name(&callback)

  

- system_version(&callback)

  

- chain_subscribe_all_heads(&callback)

  Retrieves the newest header via subscription. This will return data continuously until you unsubscribe the subscription.

  ```ruby
  subscription = client.chain_subscribe_all_heads do |data| 
    p data 
  end
  ```

- chain_unsubscribe_all_heads(subscription)

  Unsubscribe newest header subscription.

  ```ruby
  client.chain_unsubscribe_all_heads(subscription)
  ```

  

- chain_subscribe_new_heads(&callback)

  Retrieves the best header via subscription. This will return data continuously until you unsubscribe the subscription.

- chain_unsubscribe_new_heads(subscription)

  Unsubscribe the best header subscription.




- chain_subscribe_finalized_heads(&callback)

  Retrieves the best finalized header via subscription. This will return data continuously until you unsubscribe the subscription.

- chain_unsubscribe_finalized_heads(subscription)

  Unsubscribe the best finalized header subscription.

  
  
- state_subscribe_runtime_version(&callback)

  Retrieves the runtime version via subscription. 

- state_unsubscribe_runtime_version(subscription)

  Unsubscribe the runtime version subscription.

  

- state_subscribe_storage(keys, &callback)

  Subscribes to storage changes for the provided keys until unsubscribe.

  ```ruby
  subscription = client.state_subscribe_storage ["0x26aa394eea5630e07c48ae0c9558cef780d41e5e16056765bc8461851072c9d7"] do |data| 
  	p data 
  end
  ```

- state_unsubscribe_storage(subscription)

  Unsubscribe storage changes.

  
### Cutom methods based on rpc methods

These methods will encode the parameters and decode the returned data

- get_block_number(block_hash, &callback)

- get_metadata(block_hash, &callback)

- get_block(block_hash=nil, &callback)

- get_block_events(block_hash, &callback)

- subscribe_block_events(&callback, &callback)

- get_storage(module_name, storage_name, params = nil, block_hash = nil, &callback)

  ```ruby
  client.get_storage("Balances", "TotalIssuance", nil, nil) do |storage|
    p storage
  end
  
  client.get_storage("System", "Account", ["0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d"], nil) do |storage|
    p storage
  end
  
  client.get_storage("ImOnline", "AuthoredBlocks", [2818, "0x749ddc93a65dfec3af27cc7478212cb7d4b0c0357fef35a0163966ab5333b757"], nil) do |storage|
    p storage
  end
  ```

- compose_call(module_name, call_name, params, block_hash=nil, &callback)

  ```ruby
  compose_call "Balances", "Transfer", { dest: "0x586cb27c291c813ce74e86a60dad270609abf2fc8bee107e44a80ac00225c409", value: 1_000_000_000_000 }, nil do |hex|
    p hex
  end
  ```



## Synchronized client

There is also a synchronized version of the client, which is disconnected every time the data is retrieved. It is very convenient in some situations. But this version cannot use the subscription interface.

```ruby
client = SubstrateClientSync.new "wss://kusama-rpc.polkadot.io/"
p client.method_list
p client.chain_get_head
p client.chain_get_finalised_head
p client.chain_get_header(block_hash = nil)
...
```

## Docker

1. update to latest image

   `docker pull itering/substrate_client:latest`

2. Run image:

   `docker run -it itering/substrate_client:latest`

   This  will enter the container with a linux shell opened. 

   ```shell
   /usr/src/app # 
   ```

3. Type `rspec` to run all tests

   ```shell
   /usr/src/app # rspec
   ...................
   
   Finished in 0.00883 seconds (files took 0.09656 seconds to load)
   5 examples, 0 failures
   ```

4. Or, type `./bin/console` to enter the ruby interactive environment and run any decode or encode code

   ```shell
   /usr/src/app # ./bin/console
   [1] pry(main)> client = SubstrateClient.new("wss://kusama-rpc.polkadot.io/")
   => #<SubstrateClient:0x000055a78f124f58 ...
   [2] pry(main)> client.method_list do |methods| p methods end
   => ...
   [3] pry(main)> subscription = client.chain_subscribe_new_heads do |data| p data end
   ...
   ```

5. Or, SubstrateClientSync:

   ```shell
   /usr/src/app # ./bin/console
   [1] pry(main)> client = SubstrateClientSync.new("wss://kusama-rpc.polkadot.io/", spec_name: "kusama")
   => #<SubstrateClientSync:0x000055a78edfd6e0 @request_id=1, @spec_name="kusama", @url="wss://kusama-rpc.polkadot.io/">
   [2] pry(main)> client.chain_get_head
   => "0xb3c3a220d4639b7c62f179f534b3a66336a115ebc18f13db053f0c57437c45fc"
   ```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/[USERNAME]/substrate_client. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).

## Code of Conduct

Everyone interacting in the SubstrateClient project’s codebases, issue trackers, chat rooms and mailing lists is expected to follow the [code of conduct](https://github.com/[USERNAME]/substrate_client/blob/master/CODE_OF_CONDUCT.md).
