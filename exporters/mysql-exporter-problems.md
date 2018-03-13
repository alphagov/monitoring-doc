### Notes for this ticket:

https://trello.com/c/3TUdrOgD/317-demonstrate-scraping-metrics-from-backing-services-on-the-paas



We ran into quite a lot of difficulty configuring an exporter and Rails app for MySQL. These notes explain what was difficult and how we fixed it.



**mysq-exporter-paas**

https://github.com/alphagov/mysql-exporter-paas



Problem 1:

We had problems connecting to the MySQL backend. After binding the service, the PaaS provides a **DATABASE_URL** environment variable. We set this as the **DATA_SOURCE_NAME** but this didn't work. This was surprising because the Postgres exporter depends on the same environment variable name and that just worked. We wrongly assumed we could use the same environment variable.

Solution 1:

We fixed this by reading [the documenation](https://github.com/prometheus/mysqld_exporter) more closely. The **DATA_SOURCE_NAME** requires this format:

```
username:password@(host:port)/dbname?param=value
```

We wrote a [Ruby script](https://github.com/alphagov/mysql-exporter-paas/blob/master/data_source) that read the **VCAP_SERVICES** and print a URL in this format. We then exported this to the environment variable when running the exporter.





Problem 2:

Initially, we'd switched on all the 'collector settings' for the exporter to gather as many metrics as possible. When running the exporter, the process was starting, but reported `mysql_up 0` at `/metrics`. It wasn't obvious to us this meant the exporter was failing to gather metrics from MySQL.

Solution 2:

We traced the logs on the exporter `cf logs —recent <app-name>` and found it was printing hundreds of lines of warnings when starting the exporter. This was because our MySQL user wasn't priviledged enough to query these things. We switched off collector settings based on this output until it ran cleanly. This meant removing the following:

```
--collect.engine_innodb_status=true
--collect.engine_tokudb_status=true
--collect.info_schema.innodb_metrics=true
--collect.info_schema.innodb_tablespaces=true
--collect.perf_schema.eventswaits=true
--collect.perf_schema.file_events=true
--collect.perf_schema.file_instances=true
--collect.perf_schema.indexiowaits=true
--collect.perf_schema.tableiowaits=true
--collect.perf_schema.tablelocks=true
```

We also needed to explicitly disable this setting which is on by default:

```
--collect.slave_status=false
```

The impact of this means we're not collecting as many metrics as we could be. We're not sure if the metrics collected by these settings would be useful to teams. It might be that we need to speak to the GOV.UK PaaS team to ask them about priviledges as whether this is expected.



**prometheus-rails-sample-app**

https://github.com/alphagov/prometheus-rails-sample-app



Problem 1:

We [made a branch](https://github.com/alphagov/prometheus-rails-sample-app/tree/mysql-backend) with the intention of using mysql as the database backend instead of postgres. When we deployed, the application couldn't connect to MySQL. It reported `Authentication failed` and the app failed to start.

This meant it was difficult to debug because there was no server to ssh to. We changed the `manifest.yml` to run the server, even if the database migrations failed and this at least gave us a machine to connect to. We then debugged in `irb` which required setting the gem path:

```
export GEM_HOME=deps/0/vendor_bundle/ruby/2.3.0
irb -r mysql2
> Mysql::Client.new(...)
```

We then referred to the [mysql2 docs](https://github.com/brianmario/mysql2) to set the connection details, but no matter what we tried, this didn't work.

Solution 1:

We gave up on this approach and started making a new Rails app instead using the latest version of Rails.





**mysql-demo-app**

https://github.com/alphagov/mysql-demo-app



Problem 1:

After making a simple Rails app backed by MySQL, we wrote a manifest.yml to upload it to the PaaS.

This still wasn't working, but we did manage to successfully connect via the `Mysql::Client` in an `irb` session. We were missing the `sslverify: false` param, which isn't available in the old version used by the other app. We found the `config/database.yml` file written by the `cf push` process wasn't working (potentially because it wasn't writing this param).

Solution:

We wrote this file ourselves using the **VCAP_SERVICES** environment variable. We reused [a script](https://github.com/alphagov/mysql-demo-app/blob/master/write_yaml) for this.
