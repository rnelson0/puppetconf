<!SLIDE incremental>
# Blueprint
What's the high level plan to get from here to there?

* Start with Puppet 3.x
* Read the release notes
* Stairstep Roadmap
* Validate / Create rspec tests
* Refactor until your current version passes all tests
* Replace / Snapshot and Upgrade the Master
* Upgrade the Agents
* Repeat the steps for each stairstep until you are get to 4.current


~~~SECTION:notes~~~
This applies whether you're using an all-in-one or split setup. I'll try and point out where things would change between the two types. And if you're using masterless, a lot will still apply, except for the master parts, of course.

If you're on Puppet 2.x, you need to get to 3.x first! But that's not this talk. There were some good talks about this at PuppetConf 2015.

If you're actually starting new - start with Puppet 4! And then you might want to go attend another session :)
~~~ENDSECTION~~~

