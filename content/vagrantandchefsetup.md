---
Title: "Vagrant and Chef setup"
Description: "Vagrant and Chef setup"
Tags: ["abstraction","chef","cloud","configuration management","future","howto","simulator","vagrant","virtualization"]
Date: "2011-08-05"
Categories:
  - "blog"
Slug: "vagrantandchefsetup"
---

<p>I've been reading through ThoughtWorks' latest &#8216;<a href="http://www.thoughtworks.com/radar" title="radar" target="_blank">technology radar</a>&#8216; which led me to look up <a href="http://vagrantup.com/" title="Vagrant" target="_blank">Vagrant</a>, one of the tools they list as worth exploring.</p><p>Vagrant is a framework for building and deploying Virtual Machine environments, using Oracle VirtualBox for the actual VMs and utilizing Chef for configuration management.  </p><p>Watching through this intro video:</p><p>http://vimeo.com/9976342</p><p>i was quite intrigued as it is very similar to what i was looking to achieve earlier when i was <a href="http://www.theb0ardside.com/?p=22" title="Xen and Puppet" target="_blank">experimenting with installing Xen and configuring with Puppet.</a></p><p>So here's what I experienced during the setup of Vagrant on my Macbook - I decided to start with a simple Chef install to familiarise myself with Chef itself and it's own requirements CouchDB, RabbitMQ and Solr, mostly by following <a href="http://wiki.opscode.com/display/chef/Manual+Chef+Server+Configuration" title="Chef Manual Install OS X" target="_blank">these instructions</a> - </p><p><strong>-CHEF INSTALL-</strong></p><p><strong>sudo gem install chef<br />sudo gem install ohai<br /></strong></p><p>Chef uses couchDB as it's datastore, so we need to install it using the instructions <a href="http://wiki.apache.org/couchdb/Installing_on_OSX" title="CouchDB install OS X" target="_blank">here</a> </p><p><strong>brew install couchdb</strong></p><p>The instructions I list above also contains steps to install a couchDB user and set it up as a daemon. They didn't work for me, and after 30mins of troubleshooting, i gave up and went with the simpler option of running it under my own user - in production this will be running on a Linux server rather than my Macbook, so it seemed fair enough -</p><p><strong>cp /usr/local/Cellar/couchdb/1.1.0/Library/LaunchDaemons/org.apache.couchdb.plist ~/Library/LaunchAgents/</p><p>launchctl load -w ~/Library/LaunchAgents/org.apache.couchdb.plist</strong></p><p>Check its running okay by going to<br /><a href="http://127.0.0.1:5984/" title="Localhost" target="_blank">http://127.0.0.1:5984/</a></p><p>which should provide something akin to :<br /><strong>{&#8220;couchdb&#8221;:&#8221;Welcome&#8221;,&#8221;version&#8221;:&#8221;1.1.0&#8243;}</strong></p><p><strong>- INSTALL RABBITMQ -</strong></p><p><strong>brew install rabbitmq<br />/usr/local/sbin/rabbitmq-server -detached</p><p>sudo rabbitmqctl add_vhost /chef<br />sudo rabbitmqctl add_user chef testing<br />sudo rabbitmqctl set_permissions -p /chef chef &#8220;.*&#8221; &#8220;.*&#8221; &#8220;.*&#8221;<br /></strong></p><p>Ok, Gettin' back to my mission, break out the whipped cream and the cherries, then I go through all the fly positions - oh, wrong mission! </p><p>Ok..</p><p><strong>brew install gecode<br />brew install solr</p><p>sudo gem install chef-server chef-server-api chef-server chef-solr<br />sudo gem install chef-server-webui<br />sudo chef-solr-installer<br /></strong></p><p>Setup a conf file -<br /><strong>sudo mkdir /etc/chef<br />sudo vi /etc/chef/server.rb</strong> - paste in the example from:</p><p><a href="http://wiki.opscode.com/display/chef/Manual+Chef+Server+Configuration" title="chef conf" target="_blank">http://wiki.opscode.com/display/chef/Manual+Chef+Server+Configuration</a> - making the appropriate changes for your FQDN</p><p>At this point, the above instructions ask you to start the indexer however the instructions haven't been updated to reflect changes to Chef version 0.10.2 in which chef-solr-indexer has been replaced with chef-expander</p><p>So, instead of running:<br /><strong>sudo chef-solr-indexer </strong></p><p>you instead need to run:<br /><strong>sudo chef-expander -n1 -d<br /></strong></p><p>Next i tried<br /><strong>sudo chef-solr</strong></p><p>which ran into<br /><em>&#8220;`configure_chef': uninitialized constant Chef::Application::SocketError (NameError)&#8221;</em></p><p>i had to create an <strong>/etc/chef/solr.rb</strong> file and simply add this to the file:</p><p><strong>require &#8216;socket'</strong> </p><p>startup now worked -<br />if you want to daemonize it, use:</p><p><strong>sudo chef-solr -d</strong></p><p>Next start Chef Server with:<br /><strong>sudo chef-server -N -e production -d</strong></p><p>and finally:<br /><strong>sudo chef-server-webui -p 4040 -e production</strong></p><p>Now you should be up and running - you need to configure the command client &#8216;Knife' follwing the instructions <a href="http://wiki.opscode.com/display/chef/Manual+Chef+Server+Configuration" title="Knife setup" target="_blank">here</a> - under the section &#8216;<strong>Configure the Command Line Client</strong>&#8216;</p><p><strong>mkdir -p ~/.chef<br />sudo cp /etc/chef/validation.pem /etc/chef/webui.pem ~/.chef<br />sudo chown -R $USER ~/.chef</p><p>knife configure -i</strong></p><p>(follow the instructions at the link - you only need to change the location of the two pem files you copied above)</p><p>Ok, so hopefully you're at the same place as me with this all working at least as far as being able to log into CouchDB, and verifying that Chef/Knife are both working.</p><p><strong>- VAGRANT SETUP -</strong></p><p>Now, onward with the original task of Vagrant setup&#8230;<br />Have a read over the <a href="http://vagrantup.com/docs/getting-started/index.html " title="Vagrant Getting Started" target="_blank">getting started guide:</a></p><p>Install VirtualBox - download from <a href="http://www.virtualbox.org/wiki/Downloads" title="VirtualBox" target="_blank">http://www.virtualbox.org/wiki/Downloads</a></p><p>Run the installer, which should all work quite easily. Next..</p><p><strong>gem install vagrant</p><p>mkdir vagrant_guide<br />cd vagrant_guide/<br />vagrant init</strong></p><p>this creates the base Vagrantfile, which the documentation compares to a Makefile, basically a reference file for the project to work with.</p><p>Setup our first VM -<br /><strong>vagrant box add lucid32 http://files.vagrantup.com/lucid32.box</strong></p><p>This is downloaded and saved in ~/.vagrant.d/boxes/ </p><p>edit the Vagrantfile which was created and change the &#8220;box&#8221; entry to be &#8220;lucid32&#8243;, the name of the file we just saved. </p><p>Bring it online with:<br /><strong>vagrant up</strong></p><p>then ssh into with<br /><strong>vargrant ssh</strong></p><p>Ace, that worked quite easily. After a little digging around, I  logged out and tore the machine down again with<br /><strong>vagrant destroy</strong></p><p><strong>- TYING IT ALL TOGETHER -</strong><br />Now we need to connect our Vagrant install with our Chef server</p><p>First, clone the Chef repository with:<br /><strong>git clone git://github.com/opscode/chef-repo.git </strong></p><p>add this dir to your <strong>~/.chef/knife.rb</strong> file<br />i.e<br /><strong>cookbook_path           ["/Users/thorstensideboard/chef-repo/cookbooks"]</strong></p><p>Download the Vagrant cookbook they use in their examples -</p><p><strong>wget http://files.vagrantup.com/getting_started/cookbooks.tar.gz<br />tar xzvf cookbooks.tar.gz<br />mv cookbooks/* chef-repo/cookbooks/<br /></strong></p><p>Add it to our Chef server using Knife:<br /><strong>knife cookbook upload -a</strong><br />(knife uses the cookbook_path we setup above)</p><p>If you browse to your localhost at<br /><strong>http://sbd-ioda.local:4040/cookbooks/</strong><br />you should see the three new cookbooks which have been added.</p><p>Now to edit Vagrantfile and add your Chef details:<br /><code><br />Vagrant::Config.run do |config|</p><p>    config.vm.box = "lucid32"</p><p>    config.vm.provision :chef_client do |chef|</p><p>    chef.chef_server_url = "http://SBD-IODA.local:4000"<br />    chef.validation_key_path = "/Users/thorsten/.chef/validation.pem"<br />    chef.add_recipe("vagrant_main")<br />    chef.add_recipe("apt")<br />    chef.add_recipe("apache2")</p><p>    end<br />end<br /></code></p><p>I tried to load this up with<br /><strong>vagrant up</strong><br />however received:</p><blockquote><p>&#8220;[default] [Fri, 05 Aug 2011 09:27:07 -0700] INFO: *** Chef 0.10.2 ***<br />: stdout<br />[default] [Fri, 05 Aug 2011 09:27:07 -0700] INFO: Client key /etc/chef/client.pem is not present - registering<br />: stdout<br />[default] [Fri, 05 Aug 2011 09:27:28 -0700] FATAL: Stacktrace dumped to /srv/chef/file_store/chef-stacktrace.out<br />: stdout<br />[default] [Fri, 05 Aug 2011 09:27:28 -0700] FATAL: SocketError: Error connecting to http://SBD-IODA.local:4000/clients - getaddrinfo: Name or service not known&#8221;</p></blockquote><p>I figured this was a networking issue, and yeah, within the VM it has no idea of my Macbook's local hostname, which i fixed by editing its /etc/hosts file and manually adding it.</p><p>Upon issuing a<br /><strong>vagrant reload</strong>, boom! you can see the Vagrant host following the recipes and loading up a bunch of things including apache2</p><p>However at this point, you can still only access it's webserver from within the VM, so in order to access it from our own desktop browser, we can add the following line to the Vagrantfile:<br /><strong>config.vm.forward_port(&#8220;web&#8221;, 80, 8080)</strong></p><p>After another reload, you should now be able to connect to localhost:8080 and access your new VM's apache host.</p><p>In order to use this setup in any sort of dev environment will still need a good deal more work, but for the moment, this should be enough to get you up and running and able to explore both Vagrant and Chef. </p>