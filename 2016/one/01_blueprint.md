<!SLIDE incremental>
# Blueprint
What is the high level plan to get from here to there?

* Start with Puppet 3.x
* Read the release notes
* Plan your Stairstep Roadmap
* Validate / Create rspec tests
* Refactor until your current version passes all tests
* Replace / Snapshot and Upgrade the Master
* Upgrade the Agents
* Repeat the Refactor / Upgrade steps until you get to 4.latest


~~~SECTION:notes~~~
If you are on Puppet 2.x, you need to get to 3.x first! But that is not this talk. There were some good talks about this at PuppetConf 2015. If you are actually starting new - start with Puppet 4! And then you might want to go attend another session :)

It does not really change the overall blueprint if you have a single all-in-one master, a Master of Masters, or a distributed component setup - just requires a little more planning on certain steps.
~~~ENDSECTION~~~
