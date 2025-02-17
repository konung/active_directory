= Active Directory

Ruby Integration with Microsoft's Active Directory system based on original code by Justin Mecham and James Hunt at http://rubyforge.org/projects/activedirectory

See documentation on ActiveDirectory::Base for more information.

Caching:
Queries for membership and group membership are based on the distinguished name of objects.  Doing a lot of queries, especially for a Rails app, is a sizable slowdown.  To alleviate the problem, I've implemented a very basic cache for queries which search by :distinguishedname.  This is diabled by default.  All other queries are unaffected.


<h3>INSTALL (with Bundler for instance)</h3>
In Gemfile:
<pre>
gem 'active_directory', :git => 'git://github.com/richardun/active_directory.git'
</pre>

Run bundle install: <em> :; is my prompt </em>
<pre>
:; bundle install
</pre>

You can do this next part however you like, but I put the settings global variable in ../config/initializers/ad.rb...
<pre>
# Uses the same settings as net/ldap
AD_SETTINGS = {
	:host => 'domain-controller.example.local',
	:base => 'dc=example,dc=local',
	:port => 636,
	:encryption => :simple_tls,
	:auth => {
	  :method => :simple,
	  :username => "username",
	  :password => "password"
	}
}
</pre>

Then I put the base initialization in ../app/controllers/application_controller.rb...
<pre>
ActiveDirectory::Base.setup(settings)
</pre>

Then in my model, or anywhere really, I can look for AD users, etc.

<pre>
ActiveDirectory::User.find(:all)
ActiveDirectory::User.find(:first, :userprincipalname => "richard.navarrete@domain.com")

ActiveDirectory::Group.find(:all)

#Caching is disabled by default, to enable:
ActiveDirectory::Base.enable_cache
ActiveDirectory::Base.disable_cache
ActiveDirectory::Base.cache?
</pre>

You can also limit the fields that get returned, just like with ActiveRecord.

In ActiveRecord, you use ":select => ['select this', 'or', 'that']" - you can do this or use the net/ldap syntax of ":attributes => ..."  You should use one or the other, but not both.

<pre>
ad_user = ActiveDirectory::User.find(:all, :attributes => ['givenname', 'sn'])

# if you don't give it an array and just one item, that's ok too...
ad_user = ActiveDirectory::User.find(:all, :attributes => 'givenname')

puts ad_user.givenname #=> Richard
puts ad_user.sn #=> Navarrete
puts ad_user.name #=> Richard Navarrete

# But looking for a field you didn't return will raise an ArgumentError.
puts ad_user.mail #=> ArgumentError: no id given
</pre>
