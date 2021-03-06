---
title: Pantheon YAML Configuration Files
description: Learn how to use a site level configuration file to manage advanced settings per site and how to set default configurations for sites running a Custom Upstream.
tags: [pantheonyml, infrastructure]
categories: []
---
Hook into platform workflows and advanced site configurations using Pantheon's YAML configuration files, `pantheon.upstream.yml` and `pantheon.yml`. The `pantheon.upstream.yml` file is used to set default configurations for sites created from a [Custom Upstream](/docs/custom-upstream/), while the `pantheon.yml` file is used to manage configurations for one particular site. Both configuration files utilize the same set of properties that define how your Pantheon environments will behave.

When the same configuration value is defined in both files, the value from `pantheon.yml` will override the value from `pantheon.upstream.yml`.

Keeping the configuration settings defined upstream in a separate file form the configuration settings that are specific to the site prevents merge conflicts when a Custom Upstream needs to change its default Pantheon configuration.

## Custom Upstream Configurations: pantheon.upstream.yml
The `pantheon.upstream.yml` file is used when working with [Custom Upstreams](/docs/custom-upstream/). It sets default values for advanced site configurations to be used downstream.

This upstream file may contain [any of the properties described below](#advanced-site-configuration) but it cannot define any Quicksilver workflows. If a property such as `php_version` is defined upstream, then its value will be used only for sites that do not define the same property in their site level `pantheon.yml` file.

### Create and Maintain
Create the `pantheon.upstream.yml` in the root directory of an Organization's Custom Upstream. This file should only be edited in the Custom Upstream repository where it is defined. Similarly, the Custom Upstream repository should not define a `pantheon.yml` file; it should place all configuration settings in the upstream file instead.

To avoid merge conflicts from accidentally changing the `pantheon.upstream.yml` file in a site downstream, we recommended adding it to the Custom Upstream's `.gitignore` file:

```
# Avoid accidental modification of pantheon.upstream.yml in sites
# created from this upstream
pantheon.upstream.yml
```

Make changes to the file in the Custom Upstream repository and use force-add to commit and distribute downstream:

```bash
$ git add -f pantheon.upstream.yml
$ git commit -n "Change the default php version for this upstream to 7.0"
```

### Example File Contents

The `pantheon.upstream.yml` file should contain default content similar to the example file shown below:
```yaml
# IMPORTANT NOTE:
# Do not edit this file unless you are doing so in your custom upstream repository.
# Override the defaults specified here in a site-specific `pantheon.yml` file.
# For more information see: https://pantheon.io/docs/pantheon-yml
api_version: 1
php_version: 7.0
```
## Site Level Configurations: pantheon.yml
Use `pantheon.yml` when configuring a specific Pantheon site. Create a new file named `pantheon.yml` in the root of your site's code repository. Refer to [example.pantheon.yml](https://github.com/pantheon-systems/quicksilver-examples/blob/master/example.pantheon.yml) for example configurations. Refer to the [Pantheon Cloud Integration Examples](https://github.com/pantheon-systems/quicksilver-examples) repository for example scripts for use with Quicksilver Platform Hooks.

The advanced site configuration properties define how your Pantheon environments will behave.

## Advanced Site Configuration
### Required Properties
You must define the `api_version` property for the `pantheon.yml` configuration file to be valid:
```yaml
api_version: 1
```

This property must also be defined in the `pantheon.upstream.yml` file present in any upstream repository.

### Protected Web Paths

If you have files or directories inside of your docroot that you'd like to protect from public web access, use `protected_web_paths`. For example, if you add the following `pantheon.yml` to the root of your site's codebase:

```yaml
protected_web_paths:
  - /example.txt
  - /example_directory
```

That will ensure that if someone visits `https://example.com/example.txt` or `https://example.com/example_directory/any_nested_file.txt`, both will return a 403 error instead of being displayed.

**Considerations:**

* Specify the exact path; no regex or wildcards allowed
* Paths begin with a leading `/` and are relative to your docroot
* Limited to 24 protected paths
* You may not be able to protect files or paths with special characters
* Wait a few seconds for changes to take effect

### Nested Docroot

If you'd like to nest your docroot one level beneath your code repository in a directory named `web`, you may do so with the following property:

```yaml
web_docroot: true
```

The name of the nested directory is not configurable.

The `web_docroot` property should only be set in the upstream's `pantheon.upstream.yml` file. If a site stipulates a different docroot than its upstream, then it must also relocate all of its docroot files to the new location, which would prevent the site from applying future updates from its upstream. To continue to use the dashboard update feature, a site should use an upstream that already maintains a relocated document root.

For more information, see [Serving Sites from the Web Subdirectory](/docs/nested-docroot).


### PHP Version

Add `php_version` to the top level of the `pantheon.yml` file to configure the PHP version. Your PHP version will be managed in version control and deployed along with the rest of your site's code. This encourages a good best practice of testing before making a change on your Live site.

```yaml
php_version: 7.0
```
If you are upgrading to PHP 7, you may need to make changes to your site's codebase for compatibility. If you use Drupal, you need to [switch to Drush version 7 or higher](https://pantheon.io/docs/drush-versions/#configure-drush-version). For more information, see [Upgrade PHP Versions](/docs/php-versions).

In general, it is best to not set the php_version in a site's `pantheon.yml`; instead, leave php_version property out of your `pantheon.yml` file and accept the version set in the upstream's `pantheon.upstream.yml` file. This will allow the php version used in the site to automatically bump up whenever the version is changed in the upstream. If a site needs to stay on a lower version of php due to limitations of specific modules or plugins that it is using, for example, then it should define the php version it needs in its `pantheon.yml` file. If this is done, then the version of php set by the upstream will thereafter be ignored.

### Drush Version
Add `drush_version` to the top level of the `pantheon.yml` file to configure the Drush version used when making calls remotely on Pantheon:

```yaml
drush_version: 8
```
For more information, see [Managing Drush Versions on Pantheon](/docs/drush-versions).
## Quicksilver Platform Integration Hooks

Quicksilver lets you hook into platform workflows to automate your Pantheon workflow. Tell us which script you want to run, and we'll run it automatically every time you or another team member triggers the workflow operation. View (and contribute) to a [growing set of example scripts](https://github.com/pantheon-systems/quicksilver-examples/). Find examples to enable functionality like chat-ops, database sanitization, deployment logging, and automated testing operations with a CI server.

For more information, see [Automate your Workflow with Quicksilver Platform Integration Hooks](/docs/quicksilver).


## Troubleshooting

We will reject a commit that includes a `pantheon.yml` error, with a message like:
```nohighlight
remote: PANTHEON ERROR:
remote:
remote: Changes to `pantheon.yml` detected, but there was an error while processing it:
remote:
remote:
remote: Version '2' is not a valid pantheon.yml version!
remote: Valid versions are: 1
```

While our parser will reject a `pantheon.yml` that is invalid, it won't necessarily give you the exact reason the file is invalid. Please refer to the examples above for exact syntax.

### Deploying Configuration Changes to Multidev
Changes made to the `pantheon.yml` file on a branch **are not** detected when creating the Multidev environment for that branch. It is necessary to create the multidev first, and then modify pantheon.yml. As a workaround, make some modification the `pantheon.yml` file and re-commit to the Multidev environment. You will then receive a notice indicating configuration changes have been detected and applied to the Multidev environment:

```nohighlight
remote:
remote: PANTHEON NOTICE:
remote:
remote: Changes to `pantheon.yml` detected.
remote:
remote: Successfully applied `pantheon.yml` to the 'new-feature' environment.
remote:
remote:
```

## See Also
- [Automating and Integrating your Pantheon Workflow with Quicksilver Platform Hooks](/docs/quicksilver)  
- [Upgrade PHP Versions](/docs/php-versions)
