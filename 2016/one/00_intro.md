<!SLIDE center subsection>
# Enjoying the Journey from Puppet 3.x to 4.x
<center>(jfdi)</center>



<!SLIDE center>
<center>Rob Nelson</center>
<center>[Vox Pupuli](https://voxpupuli.org) and [puppet-lint](http://puppet-lint.com) Contributor</center>
<center>[@rnelson0](https://twitter.com/rnelson0), [http://rnelson0.com](http://rnelson0.com)</center>



<!SLIDE center>
# What are we talking about today?
* Upgrade our Puppet master(s) and agents to Puppet 4.x
* Refactor our code base for Puppet 4, remove Puppet 2- and 3-isms.
* Not always a linear progression, we may visit steps multiple times.



<!SLIDE incremental>

# Why?

* Puppet 4 is old! First released March, 2015.
* Or: Puppet 3 is old! EOS on [December 31, 2016](https://puppet.com/misc/puppet-enterprise-lifecycle). When the last PE 3.x is EOS, Opensource will be EOS as well.
* Lots of new things in the Puppet 4 language (3.x - Future Parser) to take advantage of. Iteration, typed variables, etc.
* Application Orchestration for PE users.
 * FOSS will eventually get AO. Some [FOSS implementations](https://github.com/ripienaar/mcollective-choria) are appearing, too.
* Modules are starting to only support Puppet 4. 
* Let EOL Ruby versions (1.8, 1.9, early 2.x) and Puppet w/Passenger go!
* Don't slow down the DevOps. Stairstep upgrades get harder and harder all the time. Stay current, it makes each upgrade minutes long instead of hours, days, or weeks.
* Puppet 5 is coming!

~~~SECTION:notes~~~
Puppet 4 isn't new anymore. But it is the most current version and you want to get on it before Puppet 5 is released.

Tons of talks on Puppet 4 language here at PuppetConf, check them out!

Puppet 3 won't turn into a pumpkin on 1/1/2017, but you may find yourself stuck between a rock and a hard place when you need a newer version of a module that requires Puppet 4, probably because it is taking advantage of the language.

Puppetserver is the future and Ruby moves fast, take advantage of the AIO builds when possible.
~~~ENDSECTION~~~



<!SLIDE center>
# Who does this apply to?
* Puppet Enterprise Users
* Puppet Opensource Users
* Masterful and Masterless setups

~~~SECTION:notes~~~
This is admittedly focused on a masterful setup, but if you have a masterless setup, a majority will still apply. We'll try and point it out as we go, and feel free to ask questions or point things out as we go.
~~~ENDSECTION~~~
