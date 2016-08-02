## Enjoying the Journey from Puppet 3.x to 4.x ##

### What are we talking about? Who does this apply to? ###

* Upgrade your Puppet master(s) and agents to Puppet 4.x.
* Refactor your code base for Puppet 4, remove Puppet 2- and 3-isms.
* Not a linear progression, you may hit each step multiple times.

### Who does this apply to? ###

* Puppet Enterprise Users
* Puppet Opensource Users

### Why? ###

* Puppet 4 is new!
* Or: Puppet 3 is old! EOS on [December 31, 2016](https://puppet.com/misc/puppet-enterprise-lifecycle). When the last PE 3.x is EOS, Opensource will be EOS as well.
* Lots of new things in the Puppet 4 language (nee Future Parser) to take advantage of. Iteration, Typed variables, etc.
* Application Orchestration!
* Modules are starting to only support Puppet 4. Puppet 3 won't turn into a pumpkin on 1/1/2017, but you may find yourself stuck between a rock and a hard place when you need a newer version of a module that requires Puppet 4, probably because it is taking advantage of the language.
* Let Ruby 1.8.7 and Puppet w/Passenger go! Puppetserver is the future and Ruby moves fast, take advantage of the AIO builds when possible.
* Don't slow down the DevOps. Stairstep upgrades get harder and harder all the time. Stay current, it makes each upgrade minutes long instead of hours, days, or weeks.

### Blueprint ###
What's the high level plan to get from here to there?

* Start with Puppet 3.x. If you're on Puppet 2.x, you need to get to 3.x first! But that's not this talk.
* Read the release notes 
* Stairstep Roadmap
* Validate/Create rspec tests.
* Refactor until your current version passes all tests.
* Snapshot and Upgrade The Master
* Upgrade the Agents
* Repeat with each stairstep until you are on the latest and greatest!

Puppet recently created some [Upgrade docs](https://docs.puppet.com/upgrade/) that go into great detail of the Upgrade process. Take time to review their guide, it's very complete and keeps being updated to match the latest version available.

Alternatively, if we practice immutable infrastructure, or want to practice it in a limited fashion, some steps can be skipped or simplified by **replacing** nodes running Puppet 3 with equivalent nodes running Puppet 4. We'll note those spots as we go.

### Release notes ###
This means all of them, not just for the latest version. If you skipped a total of 18 major/minor/patch releases, that's 18 release notes to read and consolidate in your head. If that sounds horrible, it is. As if you needed it, it's another reason to stay up to date with your systems - less reading and resolving conflicting notes!

### Stairstep Roadmap ###
This is going to vary depending on where you start, and you can make every MINOR a release even if you don't need to. For example, here are the minimum stairsteps required between 3.6.x and 4.5.x:

 * 3.6.x -> 3.8.x
 * 3.8.x w/ Future Parser
 * 3.8.x -> 4.0.0
 * 4.0.0 -> 4.latest

### Validate/Create rspec tests ###

* Without tests, you have no idea if an upgrade will be successful. Tests will not guarantee success, but can identify when failure is guaranteed.
* [puppet-module-skeleton](https://github.com/garethr/puppet-module-skeleton/blob/master/skeleton/.travis.yml) has a great rspec test setup you can copy into each of your existing modules, and can use to generate new modules.
* Have you never written an rspec-puppet test? [puppet-retrospec](https://github.com/nwops/puppet-retrospec) can help you ! It generates naive tests that may need tuned, but it's a great place to start.
* There are tons of blog posts on testing.
* Make sure your existing code passes all its test before changing it!
* Turn on Future Parser and Strict Variables as soon as possible (3.x?)

### Refactor ###

* Create a new branch against for the next stairstep version, e.g. `3.8.4`.
* Ensure you are testing against this target version in addition to your current version, e.g. `~>3.6` and `~>3.8`. (Include Gemfile examples)
* Identify failing tests, refactor as needed.
* Upgrade modules as early as possible. Be aware of the required Puppet version for a module version, and look out for defunct or migrated modules, such as those transferred to [Vox Pupuli](https://voxpupuli.org/).
* Move forward when tests are green for current and next version.

### Snapshot and Upgrade the Master ###

* Once you have confidence that your code passes all the tests and you are ready to upgrade, take a snapshot or equivalent so that you can return to a known good state properly. Take snapshots of any canary nodes you plan to use as well.
 * Alternative: Create an Upgrade environment where you can place a clone of the master and some agents and test without affecting production. Once it works in Upgrade, replace the Production master instead of upgrading it.
* Block connections to the master. If things go wrong, you may start pushing bad catalogs out and affecting agents. Even if this is in a non-production environment, you could end up breaking nodes so bad that they require manual remediation, and that's not good for anyone. You can use a firewall to block/restrict tcp port 8140, revoke certificates for non-canary nodes (OK if you have a small fleet) or revoke the CA and generate a new one and new agent certs. Lots of pros and cons to each, plan well and use what you are comfortable with.
* Upgrade the master. If anything fails, capture some logs, revert to snapshot, and unblock connections to the master, then review the failure at your leisure.
* Test the master, `puppet agent -t` against itself. If it fails, roll back.
* Test your canary nodes with `puppet agent -t`. If they fail, roll back.
* Remove the block on the master.

### Upgrade the Agents ###

* Upgrade the agents. You can do this by hand, with [puppetlabs/puppet_agent](https://forge.puppet.com/puppetlabs/puppet_agent), MCO, or any other tool at your disposal. Identify which master upgrades do not require an agent, as this is generally not required when upgrading PATCH versions and even some MINOR versions.
 * [puppet_agent requirements](https://forge.puppet.com/puppetlabs/puppet_agent/readme#setup-requirements) - `stringify_facts` defaults to true

```
[rnelson0@dns ~]$ puppet --version
3.8.7
[rnelson0@dns ~]$ sudo puppet agent -t --environment=puppet_agent
<stuff>
[rnelson0@dns ~]$ puppet --version
-bash: /usr/bin/puppet: No such file or directory
[rnelson0@dns ~]$ /opt/puppetlabs/puppet/bin/puppet --version
4.5.2
```

* This may leave other puppetlabs repos in place (i.e. `puppetlabs-release-6-11.noarch` on EL6).
* After testing is successful and a sufficient burn-in period, clean up your snapshots and any other temporary measure you put in place.

### Repeat ###

* After you confirm that a stairstep upgrade is successful, it's time to start on the next stair!
* Repeat the Refactor/Snapshot/Upgrade steps only.

### Keep up ###

* Once you're done, you're not done! Start refactoring code to leverage Puppet 4. Replace `create_resources()` with iteration. Replace `validate_*()` with typed variables. This will likely ensure fewer compatibility issues for you when Puppet 5 is available.
* PE has quarterly updates and POSS usually has more frequent updates. Try not to get more than 2 MINORs behind. Anticipate new versions by changing your Gemfile/rspec-tests to specify a puppet version of `~>4.0`, and make sure you run `bundle update` before manual tests. When 4.next is released, you'll start testing against it immediately and identify issues in advance of your upgrade.
* Aim for master upgrade times of less than an hour, it's quite possible!

### Tricks ###

Some lessons learned from POSS/PE upgrades

* PE Classifier - You can expect some changes as you move forward. You can review the [Preconfigured Node Groups documentation](https://docs.puppet.com/pe/latest/console_classes_groups_preconfigured_groups.html) yourself, but I recommend engaging Support promptly. It can be difficult to translate the classification differences into impacts to your business.
* PE 3 Bundled Ruby - If you run EL6 and want to get away from Ruby 1.8.7, it was possible to use the PE Ruby. Use rbenv/rvm instead, or update to something with a newer Ruby on the system (EL7 at least has Ruby 2.0!). [PUP-6106](https://tickets.puppetlabs.com/browse/PUP-6106)
* The [`pe_puppetserver_gem`](https://forge.puppet.com/puppetlabs/pe_puppetserver_gem) is out, [`puppetserver_gem`](https://forge.puppet.com/puppetlabs/puppetserver_gem) is in.
* PE includes support. Upgrades are a great reason to engage Support!
* Beware string conversion, in your puppet code and when values are obtained from hiera. For example, `'undef'` in rspec-puppet represents an undefined value. When Puppet runs, it's the word undef! Use `undef`, without quotes, instead. If you have a file resource with a title or path of `${undefvar}/${populatedvar}`, rspec will start failing because `file { 'undef/etc/app.conf' :}` is not valid. `'true'` vs `true` and `'false'` vs `false` is another likely candidate. Keep this in mind if tests pass but agent runs fail.
* Hiera scope
 * `%{}` in 3.x and in 4.5 resolves to the empty string. This is often used to prevent variable interpolation, as in `%%{}{environment}` to generate the string `%{environment}`. There was a regression in between those versions and it started returning the scope, giving strings like `%<#Hiera:7329A802#>{environment}`. Use `%{::}` instead, as in `%%{::}{environment}
 * Additionally, some versions expect `::` prepends to variables and others don't. This may affect your `datadir` value in `hiera.yaml`. Change `%{environment}` to `%{::environment}`.
* Don't change too much at once if you can help it. The more variables you change at once, the more difficult troubleshooting is. Try not to change your fleet's distro version during the upgrade unless you really like chasing down issues.
* Review modules and their supported versions. Some may be incorrect, or have loose assumptions (Puppet > 3). When you upgrade across major versions during refactoring, expect it to require additional troubleshooting time and effort. Generally you want to do this early, but if a particular module gives you problems, it may make sense to wait until the master upgrades are done first.
 * If the latest module reports version `999.999.999`, it is probably defunct. Check out the readme for more information!
 * There are quite a few tools to [assist with automating version upgrades in your Puppetfile](https://github.com/puppetlabs/r10k/blob/master/doc/updating-your-puppetfile.mkd#automatic-updates).
* The hiera eyaml gem will be removed during the upgrade to the 4.0 puppetserver - more accurately, the new puppetserver environment will not have it present. If you only have eyaml configured, enable the yaml backend as well and ensure that the master itself does not rely on any eyaml-only data. The first agent run on the master can redeploy the gem (with [puppet/hiera](https://forge.puppet.com/puppet/hiera) or similar) before an agent connects that requires eyaml data. This does not recur when you upgrade within the 4.x chain.

### Tools ###
The tools are constantly improving, so if you haven't looked in a while, check it out to see if your concerns were addressed.

* There are a number of catalog diff tools (including [the diffs](https://forge.puppet.com/modules?utf-8=%E2%9C%93&sort=rank&q=catalog) and [a viewer](https://github.com/camptocamp/puppet-catalog-diff-viewer)) to inspect the differences between catalogs received from different versions of Puppet.
* [puppet-ghostbuster](https://github.com/camptocamp/puppet-ghostbuster) helps you find "dead code" that you may want to prune before you start on your refactoring journey.

### Links ###
Additional information on Puppet 4 and Migrations

* [Whirlwind Tour of Puppet 4](http://www.slideshare.net/ripienaar/whirlwind-tour-of-puppet-4) by the poet, [R.I. Pienaar](https://twitter.com/ripienaar)

### Q&A ###
I am glad to try and answer any questions you have about the upgrade process. I would also like to encourage those in the audience to share their knowledge!

