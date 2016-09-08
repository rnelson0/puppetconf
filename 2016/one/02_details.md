<!SLIDE >

# Release Notes

* All of them - not just the latest version
* If you skipped a dozen major/minor/patch versions, you have lots to read and collate. If that sounds horrible, that's because it is!
* Stay up to date - less reading and resolving conflicting notes.



<!SLIDE >

# Stairstep Roadmap

Varies depending on your starting point. Find the minimum steps; you can usually skip some MINOR releases.

Enable the Future Parser before you hit 4.x.

For example, the steps between 3.6.x and 4.5.x (Open Source):

* 3.6.x -> 3.8.x
* 3.8.x w/Future Parser
* 3.8.x -> 4.5.x

~~~SECTION:notes~~~
With Open Source, you can pretty much jump from 3.8 to the latest 4.x. In fact, you probably do NOT want to go to 4.0 and then jump again, as there are bugs and config file changes and so on that may bite you.

If you are using Puppet Enterprise, however, check the KB for the upgrade stairsteps required or engage support if needed. The PE extras sometimes require intermediate PE editions to upgrade themselves before reaching your target version.
~~~ENDSECTION~~~



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

    [rnelson0@build controlrepo]$ export PUPPET_GEM_VERSION='~>3.8'
    [rnelson0@build controlrepo]$ bundle update
    Installing puppet 3.8.7 (was 4.6.0)
    [rnelson0@build controlrepo]$ bundle exec puppet --version
    3.8.7

    [rnelson0@build controlrepo]$ export PUPPET_GEM_VERSION='3.8.1'
    [rnelson0@build controlrepo]$ bundle update
    Installing puppet 3.8.1 (was 3.8.7)
    [rnelson0@build controlrepo]$ bundle exec puppet --version
    3.8.1

~~~SECTION:notes~~~
Remember to export the variable, or you'll have to prepend PUPPET_GEM_VERSION on every usage of bundle.
~~~ENDSECTION~~~



<!SLIDE incremental>

# Replace the Master

* Prepare a new operational environment - networks, etc.
* Deploy a new master on the target puppet version.
* Bootstrap your configuration/code.
* Test the master against itself, `puppet agent -t`.
* Deploy and test any canary nodes in the same operational environment.
* Collect diagnostics if it fails, repeat as necessary.

~~~SECTION:notes~~~
'Environment' is such an overloaded term. What I mean by 'operational environment' is the set of proper network, host, etc. where the new master will reside. It should NOT be in the same operational environment as the existing master, so that there is no way existing agents could check in to the new master.

There are many ways to bootstrap your master. If you don't have a process to bootstrap your master, don't worry, we'll talk about in-place upgrades shortly.
~~~ENDSECTION~~~



<!SLIDE incremental>

# Snapshot and Upgrade the Master

* Snapshot (or equivalent) the master, and any canary nodes we have.
* Restrict access to the master. Don't push bad catalogs to nodes, could cause outages.
 * Selectively block tcp/8140 with a firewall.
 * Revoke certificates for non-canary nodes, roll the snapshot back to revert.
 * Revoke the CA, generate a new CA and new agent certs (low # nodes).
 * Disable puppet agent on nodes with orchestration, ensure it doesn't turn back on in the middle of your upgrade!
* There's no right way, just find a way that works well for you.
* Upgrade the master.
* Test the master and canary nodes with `puppet agent -t`.
* Optional: Revert to snapshot, remove the block, and upgrade the master again without the block - only when you're ready to move forward.

~~~SECTION:notes~~~
There is no right way, just a way that works for you!

Remember that --noop and canary tests will send reports to puppetdb. If your puppetdb and puppet master services are on different nodes, snapshot/upgrade/revert them together.
~~~ENDSECTION~~~



<!SLIDE incremental>

# Troubleshooting

* Collect logs.
* Revert production to previous version (if applicable).
* Analyze cause.
* Refactor and remediate.
* Try again.



<!SLIDE incremental>

# Upgrade the Agents

A very non-comprehensive list of methods:

* [puppetlabs/puppet_agent](https://forge.puppet.com/puppetlabs/puppet_agent) ([requirements](https://forge.puppet.com/puppetlabs/puppet_agent/readme#setup-requirements)) allows you to update agents on their next checkin.
* MCollective or other orchestration.
* Replace nodes with new instances running the newer agent.
* By hand.
* Some combination of methods.
* You can skip this step on PATCH versions and some MINOR versions.



<!SLIDE >

# Puppet Agent in action

    @@@ Console nochrome
    [rnelson0@dns ~]$ puppet --version
    3.8.7
    [rnelson0@dns ~]$ sudo puppet agent -t --environment=puppet_agent
    <output from run>
    [rnelson0@dns ~]$ puppet --version
    -bash: /usr/bin/puppet: No such file or directory
    [rnelson0@dns ~]$ /opt/puppetlabs/puppet/bin/puppet --version
    4.5.2

There are known issues, such as leaving other puppetlabs repos in place ([MODULES-3805](https://tickets.puppetlabs.com/browse/MODULES-3805) and [3806](https://tickets.puppetlabs.com/browse/MODULES-3806)), but hard to argue with a puppetized solution!

~~~SECTION:notes~~~
Puppet 4's binary is in a new location, which makes it easy to know when the upgrade works properly! I recommend adding symlinks in your base profile if this bothers you (it can interfere with sudo's restricted path, for instance).
~~~ENDSECTION~~~



<!SLIDE >

# Repeat!

* After you're done with one upgrade, start working on the next!
* Repeat the Refactor / Snapshot / Upgrade steps only till you hit your target version.



<!SLIDE incremental >

# Keep up

<center><blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Software being &quot;Done&quot; is like lawn being &quot;Mowed&quot;.</p>&mdash; Jim Benson (@ourfounder) <a href="https://twitter.com/ourfounder/status/770075137332932608">August 29, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script></center>

Once you're done, you're not done! Refactor to take advantage of Puppet 4 language improvements, update to new tools (ex: r10k -> Code Manager for PE) and new file locations, etc.

* PE has quarterly upgrades, POSS has more frequent updates.
* Try not to get more than 2 MINORs behind.
* The less frequently you do something, the more painful it us. Upgrade early and upgrade often!
* Anticipate new versions by changing your Gemfile/rspec-tests to test against puppet version `~>4.0` (latest 4.x) and run `bundle update` before manual tests.

<!SLIDE incremental>

# Puppet 4 Language Improvements

* Replace `create_resources()` with [iteration](https://docs.puppet.com/puppet/latest/reference/lang_iteration.html).
* Replace `validate_*()` with [data types](https://docs.puppet.com/puppet/latest/reference/lang_data.html).
 * There is a `validate_legacy()` helper function coming [Real Soon Now](https://github.com/puppetlabs/puppetlabs-stdlib/pull/639) to [puppetlabs/stdlib](https://forge.puppet.com/puppetlabs/stdlib), useful when replacing `validate_*()` functions.
* Simplified resource wrappers with [*](https://docs.puppet.com/puppet/latest/reference/lang_resources_advanced.html#setting-attributes-from-a-hash) and [+](https://docs.puppet.com/puppet/latest/reference/lang_expressions.html#merging) operators.
* Improved [per-expression default attributes](https://docs.puppet.com/puppet/latest/reference/lang_resources_advanced.html#per-expression-default-attributes).
* New template type [EPP](https://docs.puppet.com/puppet/latest/reference/lang_template_epp.html) is available.
* Puppet [Lookup](https://docs.puppet.com/puppet/4.6/reference/lookup_quick.html), [Data in modules](https://docs.puppet.com/puppet/latest/reference/lookup_quick_module.html), and other hiera improvements.
* Use `$facts[]` instead of global variables to tidy up the namespace and remove ambiguity.

~~~SECTION:notes~~~
Aim for master upgrade times of <1h - it's possible!
~~~ENDSECTION~~~
