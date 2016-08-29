<!SLIDE incremental>

# Tricks, Part One

Some lessons learned from POSS/PE upgrades...

* PE includes support. Get some help planning it out.

* PE Classifier - You can expect some changes as you move forward. Review the [Preconfigured Node Groups documentation](https://docs.puppet.com/pe/latest/console_classes_groups_preconfigured_groups.html) yourself, engage support if you run into issues.

* Bundled Ruby - If you run an older distro (*cough* EL6 *cough*) and want to get away from Ruby 1.8.7, it is possible to use the PE or AIO Ruby. You can use it in a pinch, but I highly recommend you use rbenv/rvm instead, or update to something with a newer Ruby on the system (EL7 at least has Ruby 2.0!). [PUP-6106](https://tickets.puppetlabs.com/browse/PUP-6106)
 * Don't do either of these on your master. Ever!

* [`pe_puppetserver_gem`](https://forge.puppet.com/puppetlabs/pe_puppetserver_gem) is out, [`puppetserver_gem`](https://forge.puppet.com/puppetlabs/puppetserver_gem) is in.

* The hiera eyaml gem will be removed during the upgrade to the 4.0 puppetserver - more accurately, the new puppetserver environment will not have it present.
 * If you only have eyaml configured, enable the yaml backend as well and ensure that the master itself does not rely on any eyaml-only data.
 * The first agent run on the master can redeploy the gem (with [puppet/hiera](https://forge.puppet.com/puppet/hiera) or similar) before an agent connects that requires eyaml data.
 * This does not recur when you upgrade within the 4.x chain.

~~~SECTION:notes~~~
I know many of us don't like to engage support, but if you don't use it for updates, when will you use it?
~~~ENDSECTION~~~

<!SLIDE incremental>

# Tricks, Part Two

* Beware string conversion, in puppet DSL and hiera data. Understand how it works.
 * `'undef'` in rspec-puppet represents an undefined value.
 * In Puppet DSL, it's the word undef! Use `undef`, without quotes, instead.
 * If you have a file resource with a title or path of `${undefvar}/${populatedvar}`, rspec will start failing because `file { 'undef/etc/app.conf' :}` is not valid.
 * `'true'` vs `true` and `'false'` vs `false` is another likely candidate.
 * Keep this in mind if tests pass but agent runs fail.

* Hiera scope
 * `%{}` in 3.x and in 4.5 resolves to the empty string. This is often used to prevent variable interpolation, as in `%%{}{environment}` to generate the string `%{environment}`. There was a regression in between those versions and it started returning the scope, giving strings like `%<#Hiera:7329A802#>{environment}`. Use `%{::}` instead, as in `%%{::}{environment}
 * Additionally, some versions expect `::` prepends to variables and others don't. This may affect your `datadir` value in `hiera.yaml`. Change `%{environment}` to `%{::environment}`.

<!SLIDE incremental>

# Tricks, Part Three

* Review modules and their supported versions. Some may be incorrect, or have loose assumptions (`>= 3` but not `< 4`).
 * When you upgrade across major versions during refactoring, expect additional troubleshooting time and effort.
 * Generally you want to do this early, but if a particular module gives you problems, consider waiting until the master upgrades are done first.
 * If the latest module reports version `999.999.999`, it is probably defunct. Check out the readme for more information!
 * There are quite a few tools to [assist with automating version upgrades in your Puppetfile](https://github.com/puppetlabs/r10k/blob/master/doc/updating-your-puppetfile.mkd#automatic-updates).

* Don't change too much at once if you can help it. The more variables you change at once, the more difficult troubleshooting is. Try not to change your fleet's distro version during the upgrade unless you really like chasing down issues.

* There is a new [Puppet Enterprise Upgrade Service](https://m.box.com/shared_item/https%3A%2F%2Fpuppet.app.box.com%2Fs%2Fcrb2vv3z6ssc4dxhe8kwwt3yiba3jyt7) offering now! Especially helpful for PE customers on older version.

<!SLIDE incremental>

# Tools

The tools are constantly improving, so if you haven't looked in a while, check it out to see if your concerns were addressed.

* [puppetlabs_spec_helper](https://github.com/puppetlabs/puppetlabs_spec_helper), [rspec-puppet](https://github.com/rodjek/rspec-puppet/), and [puppet-lint](https://github.com/rodjek/puppet-lint/) are all improving their support for Puppet 4 on a regular basis.
* There are a number of catalog diff tools (including [the diffs](https://forge.puppet.com/modules?utf-8=%E2%9C%93&sort=rank&q=catalog) and [a viewer](https://github.com/camptocamp/puppet-catalog-diff-viewer)) to inspect the differences between catalogs received from different versions of Puppet.
* [puppet-ghostbuster](https://github.com/camptocamp/puppet-ghostbuster) helps you find "dead code" that you may want to prune before you start on your refactoring journey.
