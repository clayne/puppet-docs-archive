# Configuring and tuning Puppet Server

After you've installed Puppet Enterprise, optimize it for your environment by configuring and tuning Puppet Server settings as needed.

See the [configuration methods docs](https://puppet.com/docs/pe/latest/config_intro.html) for information on how to configure settings.

**Related information**  


[Tuning monolithic installations](tuning_monolithic.md#)

[Configure settings with Hiera](config_intro.md#)

[Increase the Java heap size for PE Java services](config_java_args.md#)

[Configure ulimit for PE services](config_ulimit.md#)

## Tune the maximum number of JRuby instances

The `jruby_max_active_instances` setting controls the maximum number of JRuby instances to allow on the Puppet Server.

### About this task

The default used in PE is the number of CPUs - 1, expressed as `$::processorcount - 1`. One instance is the minimum value and four instances is the maximum value. Four JRuby instances work for most environments.

Increasing the JRuby instances increases the amount of RAM used by `pe-puppetserver`. When you increase JRuby instances, increase the heap size. We conservatively estimate that a JRuby process uses 512MB of RAM.

### Procedure

1.  To change the number of instances using Hiera, add the following to your default `.yaml` file and set the desired number of instances:

    ```
    puppet_enterprise::master::puppetserver::jruby_max_active_instances: <number of instances>
    ```

2.  Compile and view overrides.


## Tune the Ruby load path

The `ruby_load_path` setting determines where Puppet Server finds components such as Puppet and Facter.

### About this task

The default setting is located here: `$puppetserver_jruby_puppet_ruby_load_path = ['/opt/puppetlabs/puppet/lib/ruby/vendor_ruby', '/opt/puppetlabs/puppet/cache/lib']`.

### Procedure

1.  To change the path to a different array in `pe.conf`, add the following to your `pe.conf` file on your master and set your new load path parameter:

    ```
    puppet_enterprise::master::puppetserver:: <puppetserver_jruby_puppet_ruby_load_path>
    ```

2.  Run `puppet agent -t`.


### Results

Note that if you change the `libdir` you must also change the `vardir`.

## Tune the maximum requests per JRuby instance

The `max_requests_per_instance` setting determines the maximum number of requests per instance of a JRuby interpreter before it is killed.

### About this task

The appropriate value for this parameter depends on how busy your servers are and how much you are affected by a memory leak. By default, `max_requests_per_instance` is set to 100,000 in PE.

When a JRuby interpreter is killed, all of its memory is reclaimed and it is replaced in the pool with a new interpreter. This prevents any one interpreter from consuming too much RAM, mitigating Puppet code memory leak issues and keeping Puppet Server up.

Starting a new interpreter has a performance cost, so set the parameter to get a new interpreter no more than every few hours. There are multiple interpreters running with requests balanced across them, so the lifespan of each interpreter varies.

### Procedure

1.  To increase `max_requests_per_instance` using Hiera, add the following code to your default `.yaml` and set the desired number of requests:

    ```
    puppet_enterprise::master::puppetserver::jruby_max_requests_per_instance: <number of requests>
    ```

2.  Compile and view overrides.


## Enable or disable cached data when updating classes

The optional `environment-class-cache-enabled` setting specifies whether cached data is used when updating classes in the console. When `true`, Puppet Server refreshes classes using file sync, improving performance.

### About this task

The default value for `environment-class-cache-enabled` depends on whether you use Code Manager.

-   With Code Manager, the default value is enabled \(`true`\). File sync clears the cache automatically in the background, so clearing the environment cache manually isn't required when using Code Manager.

-   Without Code Manager, the default value is disabled \(`false`\).


**Note:** If you're not using Code Manager and opt to enable this setting, make sure your code deployment method — for example r10k — clears the environment cache when it completes. If you don't clear the environment cache, the Node Classifier doesn't receive new class information until Puppet Server is restarted.

### Procedure

1.  To enable or disable cached data using Hiera, add the following to your default `.yaml`file and set the parameter to `true` or `false`.

    ```
    puppet_enterprise::master::puppetserver::
               jruby_environment_class_cache_enabled: <true OR false>
    ```

2.  Compile and view overrides.


## Changing the `environment_timeout` setting

The `environment_timeout` setting controls how long the master caches data it loads from an environment, determining how much time passes before changes to an environment's Puppet code are reflected in its environment.

In PE, the `environment_timeout` is set to `0`. This lowers the performance of your master but makes it easy for new users to deploy updated Puppet code. Once your code deployment process is mature, change this setting to `unlimited`.

**Note:** When you install Code Manager and set the `code_manager_auto_configure` parameter to `true`, `environment_timeout` is updated to `unlimited`.

Change the `environment_timeout` setting using `pe.conf,` add the following to your `pe.conf` file on your master, then run `puppet agent -t`.:

```
puppet_enterprise::master:: <environemnt_timeout>
```

For more information, see [Environments limitations](https://docs.puppet.com/puppet/5.3/environments_about.html#environments-limitations).

## Add certificates to the puppet-admin certificate whitelist

Change the `puppet-admin` certificate whitelist as needed.

### Procedure

1.  To modify whitelist certificates using `pe.conf`, insert the following in your `pe.conf` file on your master and add certificates.

    ```
    puppet_enterprise::master::puppetserver::puppet_admin_certs:
     - 'example_cert_name'
    ```

2.  Run `puppet agent -t`


## Disable update checking

Puppet Server \(pe-puppetserver\) checks for updates when it starts or restarts, and every 24 hours thereafter. It transmits basic, anonymous info to our servers at Puppet, Inc. to get update information. You can optionally turn this off.

### About this task

Specifically, it transmits:

-   Product name

-   Puppet Server version

-   IP address

-   Data collection timestamp


To turn off update checking using the console:

### Procedure

1.  Open the console, click**Classification**, and select the node group that contains the class you want to work with.

2.  On the **Configuration** tab, find the `puppet_enterprise::profile` class and find the `check_for_updates` parameter from the list and change its value to `false`.

    ```
    puppet_enterprise::profile::master::check_for_updates: false
    ```

3.  Click **Add parameter** and commit changes.

4.  On the nodes hosting the master and console, run Puppet.


## Puppet Server configuration files

At startup, Puppet Server reads all of the `.conf` files in the `conf.d` directory \(`/etc/puppetlabs/puppetserver/conf.d`\).

The `conf.d` directory contains the following files:

|File name|Description|
|---------|-----------|
|`auth.conf`|Contains authentication rules and settings for agents and API endpoint access.|
|`global.conf`|Contains global configuration settings for Puppet Server, including logging settings.|
|`metrics.conf`|Contains settings for Puppet Server metrics services.|
|`pe-puppet-server.conf`|Contains Puppet Server settings specific to Puppet Enterprise.|
|`webserver.conf`|Contains SSL and external Certificate Authority service configuration settings.|
|`ca.conf`|\(Deprecated\) Contains rules for Certificate Authority services. Superseded by `webserver.conf` and `auth.conf`.|

For information about Puppet Server configuration files, see [Puppet Server's config files](https://docs.puppet.com/puppetserver/5.1/configuration.html) and the Related topics below.

**Related information**  


[Viewing and managing Puppet Server metrics](puppet_server_metrics.md)

## pe-puppet-server.conf settings

This file contains Puppet Server settings specific to Puppet Enterprise, with all settings wrapped in a `jruby-puppet` section.

-   **`gen-home`**

    Determines where JRuby looks for gems. It is also used by the `puppetserver gem` command line tool.

    Default: `/opt/puppetlabs/puppet/cache/jruby-gems`

-   **`master-conf-dir`**

    Sets the Puppet configuration directory's path.

    Default: `/etc/puppetlabs/puppet`

-   **`master-var-dir`**

    Sets the Puppet variable directory's path.

    Default: `/opt/puppetlabs/server/data/puppetserver`

-   **`max-queued-requests`**

    Optional. Sets the maximum number of requests that may be queued waiting to borrow a from the pool. Once this limit is exceeded, a `503 Service Unavailable` response is returned for all new requests until the queue drops below the limit.

    If `max-retry-delay` is set to a positive value, then the 503 response includes a `Retry-After` header indicating a random sleep time after which the client may retry the request.

    **Note:** Don't use this solution if your managed infrastructure includes a significant number of agents older than Puppet 5.3. Older agents treat a 503 response as a failure, which ends their runs, causing groups of older agents to schedule their next runs at the same time, creating a thundering herd problem.

    Default: `0`

-   **`max-retry-delay`**

    Optional. Sets the upper limit in seconds for the random sleep set as a `Retry-After` header on 503 responses returned when `max-queued-requests` is enabled.

    Default: `1800`

-   **`jruby_max_active_instances`**

    Controls the maximum number of JRuby instances to allow on the Puppet Server.

    Default: `4`

-   **`max_requests_per_instance`**

    Sets the maximum number of requests per instance of a JRuby interpretor before it is killed.

    Default: `100000`

-   **`ruby-load-path`**

    Sets the Puppet configuration directory's path. The agent's `libdir` value is added by default.

    Default: `'/opt/puppetlabs/puppet/lib/ruby/vendor_ruby'`, `'/opt/puppetlabs/puppet/cache/lib'`


