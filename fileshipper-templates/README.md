# Icinga Director fileshipper templates

Sometimes you need to set special properties or just write Icinga 2 config DSL for some special behavior.
Sadly Icinga Director can not provide you the input to do so.

This is an example, on how to use templates with the Director, that are defined outside.

In addition this shows how to copy periods to other objects, without the need to define them everywhere.

## Fileshipper

You will need [fileshipper for Icinga Web 2](https://github.com/Icinga/icingaweb2-module-fileshipper), where you can add custom config files to the Director deployment.

Also see the [file shipping docs there](https://github.com/Icinga/icingaweb2-module-fileshipper/blob/master/doc/04-FileShipping.md).

`/etc/icingaweb2/modules/fileshipper/directories.ini`

```ini
[custom]
source = /etc/icingaweb2/modules/fileshipper/custom
target = zones.d/director-global/custom
```

## Template

In our example the template will copy `check_period` from `Host` to `Service` when set.

`/etc/icingaweb2/modules/fileshipper/custom/templates.conf`

```icinga2
template Service "branch-period" {
    // Set host for non-apply Service objects
    if (! host) {
        var host = get_host(host_name)
    }
    if (host.check_period) {
        check_period = host.check_period
    }
}
```

## Director

To create the template in Director to use it, we need to create it as disabled.

Currently the only way to do so is the CLI:

    $ icingacli director service create branch-period --object_type template --disabled
    $ icingacli director service set branch-period --notes "External object provided by fileshipper"
    $ icingacli director notification create branch-period --object_type template --disabled
    $ icingacli director notification set branch-period --vars.notes "External object provided by fileshipper"

Now you can select the template as usual, but it won't be written out by Director, that needs to come from the fileshipper now.
