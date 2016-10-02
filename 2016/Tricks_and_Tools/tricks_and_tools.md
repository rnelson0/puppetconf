<!SLIDE incremental>

# Tricks - Puppet Enterprise

Some lessons learned from POSS/PE upgrades...

* PE includes support, use it.
* Check out the [Puppet Enterprise Upgrade Service](https://m.box.com/shared_item/https%3A%2F%2Fpuppet.app.box.com%2Fs%2Fcrb2vv3z6ssc4dxhe8kwwt3yiba3jyt7)
* PE Classifier - Expect some changes as you move forward, review the [Preconfigured Node Groups documentation](https://docs.puppet.com/pe/latest/console_classes_groups_preconfigured_groups.html)
* [`pe_puppetserver_gem`](https://forge.puppet.com/puppetlabs/pe_puppetserver_gem) is out, [`puppetserver_gem`](https://forge.puppet.com/puppetlabs/puppetserver_gem) is in
* Bundled Ruby - can use it to get around old system rubies; recommend rbenv/rvm or SCL-equiv instead, or an OS/version with a newer Ruby
 * Do not ever do this on your master. EVER!

~~~SECTION:notes~~~
Now that the generalities are out of the way, we can talk specifics...

I know many of us do not like to engage support, but if you do not use it for updates, when will you use it? And if you need someone to do the upgrade, not just assistance, that is an option now as well.

Using the PE/AIO ruby can cause unexpected conflicts between bundled gems and downloaded gems. See PUP-6106 for an example.
~~~ENDSECTION~~~




<!SLIDE incremental>

# Tricks - Strings

* Beware string conversion, in puppet DSL and hiera data, understand how it works
 * rspec-puppet: `'undef'` represents an undefined value
 * Puppet DSL: it is the word undef! Try `:undef`, without quotes, instead
 * If you have a file resource with a title or path of `${undefvar}/${populatedvar}`, rspec will start failing because `file { 'undef/etc/app.conf' :}` is not valid
 * Similar issue with `'true'` vs `true` and `'false'` vs `false`
 * Other issues with input from hiera/ENC, quoted numbers as strings, etc.
 * May require acceptance tests/canary nodes to become apparent




<!SLIDE incremental>

# Tricks - Hiera

* Hiera eyaml gem is removed during the upgrade to the 4.x puppetserver
 * Enable the yaml backend and ensure that the master itself does not rely on any eyaml-only data
 * Run the agent on the master to redeploy the gem (with [puppet/hiera](https://forge.puppet.com/puppet/hiera) or similar) before agents check in.
* Hiera scope
 * `%{}`: used to prevent variable interpolation, as in `%%{}{environment}` to generate the string `%{environment}`. In 3.x and in 4.5 resolves to an empty string, in between it returned the scope, giving strings like `%<#Hiera:7329A802#>{environment}`. Use `%{::}` instead, as in `%%{::}{environment}`. Affects PE < 2016.2.0
 * `datadir`: some versions expect `::` prepends to variables and others do not. Change `%{environment}` to `%{::environment}`. Likely PE < 2016.2.0 as well

~~~SECTION:notes~~~
Anecdata suggests that sometimes eyaml needs disabled even within the 4.x series, though not in every case.

Be sure the hiera.yaml file gets put in the right location, $confdir or $codedir, depending on your puppet version.
~~~ENDSECTION~~~




<!SLIDE incremental>

# Tricks - Other

* Review modules and their supported versions. Some may be incorrect, or have loose assumptions (`>= 3` but not `< 4` and not tested against v4)
 * Upgrades across major versions means additional troubleshooting
 * Generally want to do this early, put it on the back burner if problematic
 * If the latest module reports version `999.999.999`, it is probably defunct, check out the readme for more information
 * Many tools [assist with automating version upgrades in your Puppetfile](https://github.com/puppetlabs/r10k/blob/master/doc/updating-your-puppetfile.mkd#automatic-updates)
* Minimize sizeable and entangled changes

~~~SECTION:notes~~~
"Trust but verify" support levels.

You are going to be making a lot of changes at once, but minimize coupled or entangled changes. A week of small changes beats trying to do it all at once - will likely be longer when including troubleshooting time.
~~~ENDSECTION~~~




<!SLIDE incremental>

# Tools

Tools are constantly improving, check back frequently to see if your concerns are addressed.

* [puppet-ghostbuster](https://github.com/camptocamp/puppet-ghostbuster) helps you find "dead code" that you may want to prune before you start on your refactoring journey.
* [puppetlabs_spec_helper](https://github.com/puppetlabs/puppetlabs_spec_helper), [rspec-puppet](https://github.com/rodjek/rspec-puppet/), and [puppet-lint](https://github.com/rodjek/puppet-lint/) are all improving their support for Puppet 4 on a regular basis.
* A number of catalog diff tools (including [the diffs](https://forge.puppet.com/modules?utf-8=%E2%9C%93&sort=rank&q=catalog) and [a viewer](https://github.com/camptocamp/puppet-catalog-diff-viewer)) to inspect the actual catalog differences from active nodes across different versions of Puppet.




<!SLIDE incremental>

# Links

Additional information on Puppet 4 and Migrations

* [Whirlwind Tour of Puppet 4](http://www.slideshare.net/ripienaar/whirlwind-tour-of-puppet-4) by [R.I. Pienaar](https://twitter.com/ripienaar)
* [The Power of Puppet 4](http://www.slideshare.net/tuxmea/power-of-puppet-4) by [Martin Alfke](https://twitter.com/tuxmea)
* [Puppet - our journey from Puppet 3.8 to Puppet 4](http://hggh.github.io/puppet/debian/2016/08/19/puppet-4.x.html) by [Jonas Genannt](https://twitter.com/hggh_)

