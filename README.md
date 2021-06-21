Source code for my [website](https://sdysch.github.io/), based off [reverie](https://github.com/amitmerchant1990/reverie)

# To install for local tests
* Installation instructions can be found [here](https://jekyllrb.com/docs/installation/)
* I ran into issues installing `gem install bundler jekyll`
	* Seems to be a known issue with ipv6?
	* https://stackoverflow.com/questions/49800432/gem-cannot-access-rubygems-org
* Workaround is to temporarily disable ipv6 connections
```
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.lo.disable_ipv6=1
```

## To test locally:
```
bundle exec jekyll serve
# go to http://localhost:4000/
```
