# Storage module to official 'aws-sdk' gem for Amazon S3 #

'paperclip-aws' is a full featured storage module that supports all S3 locations (US, European and Tokio) without any additional hacking.

Just install in and your model is ready to be stored all over ther world.

## Requirements ##

* [paperclip][0] ~> 2.3
* [aws-sdk][1] >= 1.0.3

## Installation ##

    gem install papepclip-aws

After this add 'papepclip-aws' to your `Gemfile` or `environment.rb`
    
## Using Storage ##

    class SomeS3Attachment < ActiveRecord::Base
      def self.s3_config
        @@s3_config ||= YAML.load(ERB.new(File.read("#{Rails.root}/config/s3.yml")).result)[Rails.env]    
      end

      has_attached_file :data,
                        :styles => {
                          :thumb => [">75x"],
                          :medium => [">600x"]
                        },                    
                        :storage => :aws,
                        :s3_credentials => {
                          :access_key_id => self.s3_config['access_key_id'],
                          :secret_access_key => self.s3_config['secret_access_key'],
                          :endpoint => self.s3_config['endpoint']
                        },
                        :s3_bucket => self.s3_config['bucket'],                    
                        :s3_host_alias => self.s3_config['s3_host_alias'],
                        :s3_acl => :public_read,
                        :s3_protocol => 'http',
                        :path => "company_documents/:id/:style/:data_file_name"  
    end
                      
## Possible options ##

### :endpoint ###
Endpoint where your bucket is located. Default is `'s3.amazonaws.com'` which is for 'US Standard' region.

You can find full list of endpoints and regions [here](http://aws.amazon.com/articles/3912#s3)

### :s3_acl  ###
Sets permissions to your objects. Values are:

    :private
    :public_read
    :public_read_write
    :authenticated_read
    :bucket_owner_read
    :bucket_owner_full_control
   
### :s3_protocol ###
Default protocol to use: `'http'` or `'https'`.

## Get your data

'paperclip-aws' redefines Paperclip `url` method to get object URL.

    def url(style=default_style, options={})
    end

Supported options are 

* `:protocol` — `'http'` or `'https'`

  Use this options to redefine default protocol, configured in model.

* `:expires`

  Sets the expiration time of the URL; after this time S3 will return an error if the URL is used.  This can be an integer (to specify the number of seconds after the current time), a string (which is parsed as a date using Time#parse), a Time, or a DateTime object. This option defaults to one hour after the current time.
  
  Default is set to 3600 seconds.

* `:action`
  
  Method, the HTTP verb or object method for which the returned URL will be valid.  Valid values:
  
  * `:get` or `:read`
  * `:put` or `:write`
  * `:delete`
  
  Default is set to `:read`, which is the most common used.

## Examples
  
Create link for file that will expire in 10 seconds after it was created. Useful when redirecting user to file.
  
    file.data.url(:original, { :expires => Time.now + 10.seconds, :protocol => 'https' })
    
    
[0]: https://github.com/thoughtbot/paperclip
[1]: https://github.com/amazonwebservices/aws-sdk-for-ruby