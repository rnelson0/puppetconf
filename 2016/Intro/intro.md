<!SLIDE center subsection>
# Enjoying the Journey from Puppet 3.x to 4.x
<center>(jfdi)</center>



<!SLIDE center>
<center>Rob Nelson</center>
<center>Contributor to [Vox Pupuli](https://voxpupuli.org), [puppet-lint](http://puppet-lint.com)</center>
<center>[@rnelson0](https://twitter.com/rnelson0), [http://rnelson0.com](http://rnelson0.com)</center>



<!SLIDE incremental>
# What are we talking about today?
* Upgrade our Puppet master(s) and agents to Puppet 4.x
* Refactor our code base for Puppet 4, remove Puppet 2- and 3-isms
* Tips, Tricks, and Tools



<!SLIDE incremental>

# Why?

* Puppet 4 is old! First released March, 2015.
* Puppet 3 is really old! End Of Support on [December 31, 2016](https://puppet.com/misc/puppet-enterprise-lifecycle).
* Puppet 4 language improvements (3.x - Future Parser).
* Application Orchestration
 * PE first, FOSS eventually. Some free implementations (such as[choria](https://github.com/ripienaar/mcollective-choria)) out there.
* Puppet 4 only modules.
* No more EOL Ruby versions (1.8, 1.9, early 2.x) or Puppet w/Passenger!
* Do not slow down the DevOps. Stairstep upgrades get harder and harder all the time, stay current.
* Puppet 5 is coming!

~~~SECTION:notes~~~
Puppet 4 is not new anymore. Get there  before Puppet 5 is released.

Tons of talks on Puppet 4 language here at PuppetConf, check them out!

Puppet 3 will not turn into a pumpkin on 1/1/2017, but you may find yourself stuck between a rock and a hard place when you need a newer version of a module that requires Puppet 4 language constructs.

Puppetserver is the future and Ruby moves fast, take advantage of the AIO builds when possible.
~~~ENDSECTION~~~



<!SLIDE >
# Who does this apply to?
* Puppet Enterprise Users
* Puppet Opensource Users
* Masterful and Masterless setups

~~~SECTION:notes~~~
The talk is mostly focused on a masterful setup, but if you have a masterless setup, a most will still apply.
~~~ENDSECTION~~~
