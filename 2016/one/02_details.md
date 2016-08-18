<!SLIDE >

# Release Notes

* All of them - not just the latest version
* If you skipped a dozen major/minor/patch versions, you have lots to read and collate. If that sounds horrible, that's because it is!
* Stay up to date - less reading and resolving conflicting notes.



<!SLIDE >

# Stairstep Roadmap

Varies depending on your starting point. Find the minimum steps; you can usually skip some MINOR releases.

Enable the Future Parser before you hit 4.0.

For example, the steps between 3.6.x and 4.5.x:

* 3.6.x -> 3.8.x
* 3.8.x w/Future Parser
* 3.8.x -> 4.0.0
* 4.0.0 -> 4.latest



<!SLIDE incremental>

# Validate / Create rspec tests

* No tests, no assured behavior!
* Generate modules with [puppet-module-skeleton](https://github.com/garethr/puppet-module-skeleton/blob/master/skeleton/.travis.yml) and you get a free rspec setup, too.
* Never written an rspec-puppet test? [puppet-retrospec](https://github.com/nwops/puppet-retrospec) can help! Generates naive tests that need tuned, but a great place to start.
* Tons of blog posts on testing.
* Make sure your existing tests pass before changing the code.
* Turn on Future Parser and Strict Variables as soon as you hit 3.8.x
 * You can do it earlier, but some versions have bugs.


~~~SECTION:notes~~~
Tests do not guarantee success, but can identify many regressions and determine when failure is guaranteed.

Rspec testing can be tricky, no lie. But it doesn't have to be. Start simple and grow from there. It's worth the effort.
~~~ENDSECTION~~~

<!SLIDE >

# Example rspec tests

    @@@ Console nochrome
    $ cat spec/classes/apache_spec.rb
    require 'spec_helper'
    describe 'profile::apache', :type => :class do
      let :facts do
        {
          facts_hash
        }
      end
    
      context 'with defaults for all parameters' do
        it { is_expected.to create_class('profile::apache') }
        it { is_expected.to contain_package('httpd') }
        it { is_expected.to contain_user("apache") }
      end
    end

    [rnelson0@build03 profile:production]$ bundle exec rspec spec/classes/apache_spec.rb
    
    profile::apache
      with defaults for all parameters
        should contain Class[profile::apache]
        should contain Package[httpd]
        should contain User[apache]

    Finished in 7.82 seconds (files took 2.49 seconds to load)
    3 examples, 0 failures



<!SLIDE incremental>

# Refactor

* Create a new branch against for the next stairstep version, e.g. `3.8.6`.
* Test against this target version in addition to your current version, e.g. `~>3.6` and `~>3.8`.
* Identify failing tests, refactor as needed.
* Upgrade modules as early as possible. Be aware of the required Puppet version for a module version, and look out for defunct or migrated modules, such as those transferred to [Vox Pupuli](https://voxpupuli.org/).
* Move forward when tests are green for current and next version.



<!SLIDE >

# Testing with particular Puppet versions

    @@@ Console nochrome
    $ grep PUPPET Gemfile
      gem "puppet", ENV['PUPPET_GEM_VERSION'] || '~> 4.0'gem "puppet", ENV['PUPPET_GEM_VERSION'] || '~> 4.0'

    [rnelson0@build controlrepo]$ export PUPPET_GEM_VERSION='~>3.8'; bundle update
    Installing puppet 3.8.7 (was 4.6.0)
    [rnelson0@build controlrepo]$ bundle exec puppet --version
    3.8.7

    [rnelson0@build controlrepo]$ export PUPPET_GEM_VERSION='3.8.1'; bundle update
    Installing puppet 3.8.1 (was 3.8.7)
    [rnelson0@build controlrepo]$ bundle exec puppet --version
    3.8.1

~~~SECTION:notes~~~
Remember to export the variable, or you'll have to prepend PUPPET_GEM_VERSION on every usage of bundle.
~~~ENDSECTION~~~
