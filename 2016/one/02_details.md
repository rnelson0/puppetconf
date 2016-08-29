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
* Determine what kinds of tests you need - unit, acceptance, integration, more?
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
 * Version 999.999.999 means defunct (more later).
* Move forward when tests are green for current and next version.



<!SLIDE >

# Testing with particular Puppet versions

    @@@ Console nochrome
    $ grep PUPPET Gemfile
      gem "puppet", ENV['PUPPET_GEM_VERSION'] || '~> 4.0'

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

<!SLIDE incremental>
# Snapshot and Upgrade the Master

Not all upgrades will go well. Make sure we have a way back!

* Snapshot (or equivalent) the master, and any canary nodes we have.
* Restrict access to the master. Don't push bad catalogs to nodes, could cause outages.
 * Set up an exact duplicate environment and upgrade there.
 * Selectively block tcp/8140 with a firewall.
 * Revoke certificates for non-canary nodes (roll the snapshot back to revert).
 * Revoke the CA, generate a new CA and new agent certs.
 * Disable puppet agent on nodes with orchestration, ensure it doesn't turn back on in the middle of your upgrade!
* There's no right way, just find a way that works well for you.
* Upgrade the master.
* Test the master against itself, `puppet agent -t`.
* Test the canary nodes with `puppet agent -t` as well.
* If anything fails, collect logs, revert to your snapshots, unblock connection to the master, and go back to the *Refactor* steps to fix it before trying again.
* Optional: Revert to snapshot, remove the block, and upgrade the master again without the block - only when you're ready to move forward.
* There's no right way, just a way that works for you!

<!SLIDE incremental>

# Upgrade the Agents

A very non-complete list of methods:

* [puppetlabs/puppet_agent](https://forge.puppet.com/puppetlabs/puppet_agent) ([requirements](https://forge.puppet.com/puppetlabs/puppet_agent/readme#setup-requirements)) allows you to update agents on their next checkin.
* MCollective or other orchestration.
* Replace nodes with new instances running the newer agent.
* By hand.
* Some combination of methods.
* You can skip this step on PATCH versions and some MINOR versions

<!SLIDE >

# Puppet Agent in action

    @@@ Console nochrome
    [rnelson0@dns ~]$ puppet --version
    3.8.7
    [rnelson0@dns ~]$ sudo puppet agent -t --environment=puppet_agent
    <run output>
    -bash: /usr/bin/puppet: No such file or directory
    [rnelson0@dns ~]$ /opt/puppetlabs/puppet/bin/puppet --version
    4.5.2

There are known issues, such as leaving other puppetlabs repos in place ([MODULES-3805](https://tickets.puppetlabs.com/browse/MODULES-3805) and [3806](https://tickets.puppetlabs.com/browse/MODULES-3806)), but hard to argue with a puppetized solution!

~~~SECTION:notes~~~
Puppet 4's binary is in a new location, which makes it easy to know when the upgrade works properly!
~~~ENDSECTION~~~

<!SLIDE >

# Repeat!

* After you're done with one upgrade, start working on the next!
* Repeat the Refactor / Snapshot / Upgrade steps only till you hit your target version.

<!SLIDE incremental>

# Keep Up

* Once you're done, you're not done! Start refactoring to take advantage of Puppet 4 language improvements including, but not limited to:
 * Replace `create_resources()` with [iteration](https://docs.puppet.com/puppet/latest/reference/lang_iteration.html).
 * Replace `validate_*()` with [data types](https://docs.puppet.com/puppet/latest/reference/lang_data.html).
 * Simplified resource wrappers with [*](https://docs.puppet.com/puppet/latest/reference/lang_resources_advanced.html#setting-attributes-from-a-hash) and [+](https://docs.puppet.com/puppet/latest/reference/lang_expressions.html#merging) operators.
 * Improved [per-expression default attributes](https://docs.puppet.com/puppet/latest/reference/lang_resources_advanced.html#per-expression-default-attributes).

* PE has quarterly upgrades, POSS has more frequent updates.
 * Try not to get more than 2 MINORs behind.
 * The less frequently you do something, the more painful it us. Upgrade early and upgrade often!
 * Anticipate new versions by changing your Gemfile/rspec-tests to test against puppet version `~>4.0` (latest 4.x) and run `bundle update` before manual tests. Your test setup won't be surprised when `v4.next` is released.

* Aim for master upgrade times of <1h - it's possible!
