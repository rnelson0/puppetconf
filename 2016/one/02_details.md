<!SLIDE >

# Release Notes

* All of them - not just the latest version.
* Lots to read if you skipped a dozen major/minor/patch versions. Look for conflicts.
* Stay up to date - less reading and resolving conflicting notes.
* Your findings inform your version stairstep plans.



<!SLIDE incremental>

# Stairstep Roadmap

* Varies depending on your starting point.
* Find the minimum steps; you can usually skip some MINOR releases.
* Enable the Future Parser before you hit 4.x.
* PE: Requires intermediate versions or not-in-place migration
* FOSS: Go straight to 4.latest

~~~SECTION:notes~~~
With Open Source, you can pretty much jump from 3.8 to the latest 4.x. In fact, you probably do NOT want to go to 4.0 and then jump again, as there are bugs and config file changes and so on that may bite you.

If you are using Puppet Enterprise, however, check the KB/release notes for the upgrade stairsteps required or engage support if needed. The PE extras sometimes require intermediate PE editions to upgrade themselves before reaching your target version.
~~~ENDSECTION~~~



<!SLIDE incremental>
# FOSS Example
 
* 3.6.x -> 3.8.x
* 3.8.x w/Future Parser
* 3.8.x -> 4.latest




<!SLIDE incremental>
# PE Example

* 3.7.2 -> 3.8.6
* 3.8.3 w/Future Parser
* 3.8.3 -> 2015.3.3
* 2015.3.3 -> 2016.2.1



<!SLIDE incremental>

# Validate / Create tests

* No tests, no assured behavior!
* Generate modules with [puppet-module-skeleton](https://github.com/garethr/puppet-module-skeleton/blob/master/skeleton/.travis.yml) and you get a free rspec setup, too.
* Never written an rspec-puppet test? [puppet-retrospec](https://github.com/nwops/puppet-retrospec) can help! Generates naive tests that need tuned.
* Determine what kinds of tests you need - unit, acceptance, integration, more?
* Make sure your existing tests pass before changing the code.
* Turn on Future Parser and Strict Variables as soon as you hit 3.8.x
* Catalog Diffs - beyond rspec tests

~~~SECTION:notes~~~
If you do not have any tests yet, please attend a testing talk! "Turning Pain Into Gain" at 2:30PM today and "The Future of Testing Puppet Code" tomorrow at 3:45PM.

Tests do not guarantee success, but can identify many regressions and determine when failure is guaranteed.

Future parser existed in earlier versions but had some bugs, wait till you hit 3.8.

Rspec testing can be tricky, no lie. But it does not have to be. Start simple and grow from there. It is worth the effort.

Catalog diffs can be useful for many people, but not all. Rspec tests might be all you need!
~~~ENDSECTION~~~



<!SLIDE >

# Rspec Tests
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




<!SLIDE >

# Rspec Run
    @@@ Console nochrome
    [rnelson0@build03 profile:production]$ bundle exec rspec spec/classes/apache_spec.rb
    profile::apache
      with defaults for all parameters
        should contain Class[profile::apache]
        should contain Package[httpd]
        should contain User[apache]

    Finished in 7.82 seconds (files took 2.49 seconds to load)
    3 examples, 0 failures

~~~SECTION:notes~~~
This is a test on a profile module, not a component module, but this is absolutely needed! This is a majority of the code you are going to write and it helps with your design phase if you write them first.
~~~ENDSECTION~~~



<!SLIDE incremental>

# Refactor

* Create a new branch for the target version, e.g. `3.8.6`
* Test against current and target versions, e.g. `~>3.6` and `~>3.8`
* Identify failing tests, refactor to fix
* Upgrade modules as early as possible. Be aware of the required Puppet version for a module version, and look out for defunct or migrated modules, such as those transferred to [Vox Pupuli](https://voxpupuli.org/).
 * Version 999.999.999 means defunct (more later)
* Move forward when tests are green for current and next version




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
Remember to export the variable, or you will have to prepend PUPPET_GEM_VERSION on every usage of bundle.
~~~ENDSECTION~~~




<!SLIDE incremental>

# Replace the Master

* Prepare a new operational environment - networks, etc.
* Deploy a new master on the target puppet version.
* Bootstrap your configuration/code.
* Test the master against itself, `puppet agent -t`.
* Deploy and test any canary nodes in the same operational environment.

~~~SECTION:notes~~~
'Environment' is such an overloaded term. What I mean by 'operational environment' is the set of proper network, host, etc. where the new master will reside. It should NOT be in the same operational environment as the existing master, so that there is no way existing agents could check in to the new master.

Of course, it is fairly privileged to assume you can do this. If you do not have that luxury, do your best!

There are many ways to bootstrap your master. If you do not have a process to bootstrap your master, do not worry, we will talk about in-place upgrades shortly.
~~~ENDSECTION~~~



<!SLIDE incremental>

# Snapshot and Upgrade the Master

* Snapshot (or equivalent) the master and canary nodes
* Restrict access to the master, prevent serving bad catalogs
 * Control access with firewall/load balancer
 * Revoke certificates for non-canary nodes
 * Revoke the CA, generate a new CA and new agent certs (low # nodes)
 * Disable puppet agent on nodes with orchestration
* Upgrade the master
* Test the master and canary nodes with `puppet agent -t`
* Optional: Revert to snapshot, remove the block, and upgrade the master again without the block

~~~SECTION:notes~~~
There is no right way, just a way that works for you!

Remember that --noop and canary tests will send reports to puppetdb. If your puppetdb and puppet master services are on different nodes, snapshot/upgrade/revert them together.
~~~ENDSECTION~~~




<!SLIDE incremental>

# Troubleshooting

* Collect logs from the master and canaries
* Revert your production environment
* Analyze cause
* Refactor and remediate your code and data
* Try again
* Learn from failures, prevent them in the future




<!SLIDE incremental>

# Upgrade the Agents

A very non-comprehensive list of methods:

* [puppetlabs/puppet_agent](https://forge.puppet.com/puppetlabs/puppet_agent) ([requirements](https://forge.puppet.com/puppetlabs/puppet_agent/readme#setup-requirements)) allows you to update agents on their next checkin
* Orchestration
* Replace nodes with new instances running the newer agent
* By hand
* Can often skip this step on PATCH versions and some MINOR versions (see release notes)




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

~~~SECTION:notes~~~
There are known issues, such as leaving other puppetlabs repos in place ([MODULES-3805](https://tickets.puppetlabs.com/browse/MODULES-3805) and [3806](https://tickets.puppetlabs.com/browse/MODULES-3806)), but hard to argue with a puppetized solution!

Puppet 4 is binary is in a new location, which makes it easy to know when the upgrade works properly! I recommend adding symlinks in your base profile if this bothers you (it can interfere with the sudo restricted path, for instance).
~~~ENDSECTION~~~




<!SLIDE >

# Repeat!

* After you are done with one upgrade, start working on the next!
* Repeat the Refactor / Upgrade steps till you hit your target version



<!SLIDE incremental >

# Keeping up

<center><blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Software being &quot;Done&quot; is like lawn being &quot;Mowed&quot;.</p>&mdash; Jim Benson (@ourfounder) <a href="https://twitter.com/ourfounder/status/770075137332932608">August 29, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script></center>

Once you're done, you're not done! Refactor to take advantage of Puppet 4 language improvements, update to new tools (ex: r10k -> Code Manager for PE) and new file locations, etc.

* PE has quarterly upgrades, POSS more frequent
* The less frequently you do something, the more painful it us. Upgrade early and upgrade often!
* Try not to get more than 2 MINORs behind
* Anticipate new versions by changing your Gemfile/rspec-tests to test against puppet version `~>4.0` (latest v4) and run `bundle update` before manual tests




<!SLIDE incremental>

# Puppet 4 Language Improvements

* Replace `create_resources()` with [iteration](https://docs.puppet.com/puppet/latest/reference/lang_iteration.html)
* Replace `validate_*()` with [data types](https://docs.puppet.com/puppet/latest/reference/lang_data.html)
 * There is a `validate_legacy()` helper function coming [Real Soon Now](https://github.com/puppetlabs/puppetlabs-stdlib/pull/639) to [puppetlabs/stdlib](https://forge.puppet.com/puppetlabs/stdlib), useful when replacing `validate_*()` functions
* Simplified resource wrappers with [*](https://docs.puppet.com/puppet/latest/reference/lang_resources_advanced.html#setting-attributes-from-a-hash) and [+](https://docs.puppet.com/puppet/latest/reference/lang_expressions.html#merging) operators
* Improved [per-expression default attributes](https://docs.puppet.com/puppet/latest/reference/lang_resources_advanced.html#per-expression-default-attributes)
* New template type [EPP](https://docs.puppet.com/puppet/latest/reference/lang_template_epp.html) is available
* Puppet [Lookup](https://docs.puppet.com/puppet/4.6/reference/lookup_quick.html), [Data in modules](https://docs.puppet.com/puppet/latest/reference/lookup_quick_module.html), and other hiera improvements
* Use `$facts[]` instead of global variables to tidy up the namespace and remove ambiguity

~~~SECTION:notes~~~
Aim for master upgrade times of <1h - it is possible!
~~~ENDSECTION~~~
