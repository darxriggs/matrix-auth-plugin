# Matrix Authorization Strategy Plugin

[![Build Status](https://ci.jenkins.io/job/Plugins/job/matrix-auth-plugin/job/master/badge/icon.svg)](https://ci.jenkins.io/job/Plugins/job/matrix-auth-plugin/job/master/)
[![Jenkins Plugin](https://img.shields.io/jenkins/plugin/v/matrix-auth.svg)](https://plugins.jenkins.io/matrix-auth)
[![Jenkins Plugin Installs](https://img.shields.io/jenkins/plugin/i/matrix-auth.svg)](https://plugins.jenkins.io/matrix-auth)

Implement fine-grained access control in Jenkins with this plugin.

For a basic introduction, see [the section on Matrix Authorization in the Jenkins handbook](https://jenkins.io/doc/book/managing/security/#authorization).

## Use Cases

Matrix Authorization allows configuring the lowest level permissions, such as starting new builds, configuring items, or deleting them, individually.

### Project-based configuration

Project-based matrix authorization allows configuring permissions for each item or agent independently.
Permission applying to such items or agents that are granted in the global configuration apply to all of them, unless they don't inherit global permissions (see below).

### Permission inheritance

With project-based matrix authorization, permissions are by inherited from the global configuration and any parent entities (e.g. the folder a job is in) by default.
This can be changed.
Depending on the entity being configured, all or a subset of the following _inheritance strategies_ are available:

* Inherit permissions:
  This is the default behavior.
  Permissions explicitly granted on individual items or agents will only add to permissions defined globally or in any parent items.
* Inherit global configuration only:
  This will only inherit permissions granted globally, but not those granted on parent folders.
  This way, jobs in folders can control access independently from their parent folder.
* Do not inherit permissions:
  The most restrictive inheritance configuration.
  Only permissions defined explicitly on this agent or item will be granted.
  The only exception is Overall/Administer:
  It is not possible to remove access to an agent or item from Jenkins administrators.

### Configuration as Code and Job DSL support

Matrix Authorization Strategy Plugin has full support for use in Configuration as Code and Job DSL.

For an example combining the two, see [this `configuration-as-code.yml` test resource](https://github.com/jenkinsci/matrix-auth-plugin/blob/master/src/test/resources/org/jenkinsci/plugins/matrixauth/integrations/casc/configuration-as-code.yml).


## Caveats

When using project-based matrix authorization, users granted permission to configure items or agents will be able to grant themselves all other permissions on the item or agent.
These would be inherited unless specifically disabled.

Beyond the above, administrators implementing fine-grained permissions control need to be aware of interactions between permissions, and certain overlap between them.
Some examples:

* A user not granted read access to Jenkins in general will not be able to use most of the other permissions they've been granted -- likely none of them.
* A user not granted read access to a job will not be able to start new builds, delete the job, configure the job, etc.
* When using global matrix authorization, users granted permission to configure jobs but not start them will still be able to configure the job to be periodically executed.
* Some permissions imply others.
  Most notably, Overall/Administer implies (almost) all other permissions, but other implications exist:
  For example, Job/Read implies Job/Discover.
  Descriptions for permissions will note when a permission is either implied by a permission other than Overall/Administer, or when it is not implied by any other permission.

## Changelog

### Version 2.4.2 (2019-05-02)

* [JENKINS-57313](https://issues.jenkins-ci.org/browse/JENKINS-57313): Fix a bug introduced in 2.4 that could result in exception error messages shown on the configuration page when permissions are assigned to valid user accounts that have never logged in to Jenkins.

### Version 2.4.1 (2019-04-27)

* Fix a bug introduced in 2.4 that could prevent agent configurations from being loaded

### Version 2.4 (2019-04-24)

* Increase core dependency from 2.60.1 to 2.138.3
* Configuration as Code compatibility: Integrate configurators for global and agent permissions.
* Job DSL compatibility: Add support for configuring folder permission inheritance using `authorizationMatrix` symbol
* Job DSL compatibility: Allow setting permissions using user-friendly names like _Overall/Read_
* Fix a minor UI glitch on job configuration pages

### Version 2.3 (2018-07-10)

* [JENKINS-52167](https://issues.jenkins-ci.org/browse/JENKINS-52167): Rotate column headers in Google Chrome

* [JENKINS-47424](https://issues.jenkins-ci.org/browse/JENKINS-47424): Don't show 'Implied by' note for the Overall/Administer permission
* [JENKINS-28668](https://issues.jenkins-ci.org/browse/JENKINS-28668): Use a modal dialog to add users/groups to the list to prevent accidental form submissions

### Version 2.2 (2017-11-12)

* [JENKINS-47885](https://issues.jenkins-ci.org/browse/JENKINS-47885): Work around a JavaScript error in the Configure Jenkins form when the Kubernetes plugin is installed.
* Improve performance of permission checks for internal SYSTEM user.

### Version 2.1.1 (2017-11-02)

* Do not show a warning on Jenkins startup when the Folders Plugin is not installed.

### Version 2.1 (2017-10-12)

* [JENKINS-47412](https://issues.jenkins-ci.org/browse/JENKINS-47412): Fix a bug introduced in 2.0 that prevented creation of new agents via the UI.

### Version 2.0 (2017-10-09)

**Note for users of version 2.0-beta-3: There have been no changes since that release.**

#### Important upgrade notes

* This release requires **Jenkins 2.60.1** or newer as it makes extensive use of Java 8 features (and there's currently no way to declare a minimum needed Java version other than to depend on a core that requires that Java release).
* This release uses a **new on-disk format** for permissions inheritance options. Existing options will be retained when upgrading, but **do****wngrading to older versions may result in failures to load job or folder permission data, or different (typically additional) permissions being granted after the downgrade.**
* Support for loading permissions last saved before Jenkins 1.300 (April 2009) has been dropped from this release.

#### Notable changes since 1.7

* **Flexible permission inheritance options**
    * This replaces the 'blocks inheritance' feature implemented in version 1.2\. **The on-disk storage format has changed to support this.**
    * Ensure that even "blocking inheritance" does not block administrator access. ([JENKINS-24878](https://issues.jenkins-ci.org/browse/JENKINS-24878))
    * Improve wording of inheritance options and include inline explanation about the effects. ([JENKINS-39409](https://issues.jenkins-ci.org/browse/JENKINS-39409))
* **Allow configuring per-agent permissions.** This allows e.g. restricting per-agent build permissions when using the Authorize Project plugin ([JENKINS-46654](https://issues.jenkins-ci.org/browse/JENKINS-46654))
* **Prevent accidental lockouts and unexpected lack of permissions**  
    * Improvement: When submitting a global matrix auth configuration that does not specify an administrator (often happening in accidental/premature form submissions), give the submitting user Administer permission. Note that this could mean that the 'anonymous' may still have admin permission if the form is submitted as an anonymous user. ([JENKINS-46832](https://issues.jenkins-ci.org/browse/JENKINS-46832) / [JENKINS-10871](https://issues.jenkins-ci.org/browse/JENKINS-10871))
    * Bug: Ensure that users creating a new job, folder, or node have read and configure access when using the project-based matrix authorization strategy. (<span class="js-issue-title">JENKINS-5277</span>)
    * Bug: Save the global security configuration after granting administer permission to the first user to sign up. ([JENKINS-20520](https://issues.jenkins-ci.org/browse/JENKINS-20520))
    * Bug: Ensure 'empty' matrix permission configurations can be loaded in case this is needed (e.g. programmatically defined). The fix for [JENKINS-10871](https://issues.jenkins-ci.org/browse/JENKINS-10871) will prevent this from happening accidentally. (JENKINS-9774)
    * Bug: When using container-based authentication and project-based matrix authorization, permissions granted to groups in items inside folders only may not have been granted to members of those groups.
* **UX improvements for the matrix configuration table**  
    * Improvement: Indicate whether a permission is implied by another permission in the tool tip, and also indicate when a permission is not implied by Overall/Administer (which is unusual). ([JENKINS-32506](https://issues.jenkins-ci.org/browse/JENKINS-32506))
    * Improvement: Show the full name of the user, if found, instead of the user ID. The user ID is available in the tool tip. ([JENKINS-14563](https://issues.jenkins-ci.org/browse/JENKINS-14563))
    * Improvement: Always list the 'authenticated' group, list it and 'anonymous' first, and give both of them friendly localizable display names ([JENKINS-30495](https://issues.jenkins-ci.org/browse/JENKINS-30495))
    * Improvement: Improve usability of large permission tables: Add tool tips for permission checkboxes indicating the user ID and permission involved, and add tool tips indicating affected user/group for the actions to the right of table rows. ([JENKINS-26824](https://issues.jenkins-ci.org/browse/JENKINS-26824))
* **Add support for use in the `properties()` pipeline step. For usage example, see the snippet generator.** ([JENKINS-34616](https://issues.jenkins-ci.org/browse/JENKINS-34616))
* Bug: Support case sensitivity for per-folder permissions as well, was missed in 1.7\. ([JENKINS-23805](https://issues.jenkins-ci.org/browse/JENKINS-23805))
* Bug: Prevent `NullPointerException` getting logged when a matrix auth config form is viewed. ([JENKINS-46190](https://issues.jenkins-ci.org/browse/JENKINS-46190))
* Use PNG icons with transparent background rather than GIF with white background.
* Major internal cleanup and code simplification
    * Drop support for data migration (Item.Read permission) from Jenkins 1.300 and earlier
    * Drop support for loading project-based matrix permissions last saved before September 2008

### Version 2.0-beta-3 (2017-09-20)

#### Notable changes in this release

* New Feature: Add support for use in the `properties()` pipeline step. For usage example, see the snippet generator. ([JENKINS-34616](https://issues.jenkins-ci.org/browse/JENKINS-34616))

### Version 2.0-beta-2 (2017-09-19)

#### Notable changes in this release

* Fix regression in 2.0-beta-1 that broke compatibility with Role-based Authorization Strategy Plugin (role-strategy). ([JENKINS-46923](https://issues.jenkins-ci.org/browse/JENKINS-46923))
* Fix regression in 2.0-beta-1 that made permission tool tips disappear in job, folder, and node property configuration forms.
* Fix regression in 2.0-beta-1 that showed permission group table cells in the config form for groups that did not apply to the current job, folder, or node property.
* Use PNG icons with transparent background rather than GIF with white background.
* Fix label of node property introduced in 2.0-beta-1.
* Show user names in tooltips for checkboxes and buttons in newly added rows.
* Internal refactoring to reduce code duplication.
* Restrict external use of some APIs newly introduced since 1.7.

### Version 2.0-beta-1 (2017-09-16)

#### Notable changes in this release

* Most of the features and fixes that made it into version 2.0.

### Version 1.7 (Jun 28, 2017)

* [JENKINS-44665](https://issues.jenkins-ci.org/browse/JENKINS-44665) Select All/None buttons rather than a button to invert.

* [JENKINS-23805](https://issues.jenkins-ci.org/browse/JENKINS-23805) Support case sensitivity modes of the security realm.

### Version 1.6 (May 18, 2017)

* [JENKINS-29815](https://issues.jenkins-ci.org/browse/JENKINS-29815) Add the same tick-box to disable inheritance of global permissions to Folders as already existed for Projects.

### Version 1.5 (Apr 10, 2017)

* [SECURITY-410](https://jenkins.io/security/advisory/2017-04-10/#matrix-authorization-strategy-plugin-allowed-configuring-dangerous-permissions): plugin allowed configuration of dangerous permissions. See advisory for details.

### Version 1.4 (May 24, 2016)

* Stack trace displayed on startup with Folders plugin disabled or missing.
* Better display of unrecognized usernames in configuration matrix.

### Version 1.3.2 (Feb 25, 2016)

* Stack trace displayed when attempting to configure authorization property on a folder.

### Version 1.3.1 (Feb 25, 2016)

* Moved forgotten resource from the Folders plugin. Also now forces the Icon Shim update.

### Version 1.3 (Feb 22, 2016)

* Inverted dependency so this plugin now depends on the [CloudBees Folders Plugin](/display/JENKINS/CloudBees+Folders+Plugin). **If you accept this update, you must also update the** **[Icon Shim Plugin](/display/JENKINS/Icon+Shim+Plugin)** **(to 2.0.3 or later).**
* Extended diagnostic fix made in 1.1.
* Silently ignore unknown permissions instead of throwing an `IllegalArgumentException`.
* [JENKINS-29527](https://issues.jenkins-ci.org/browse/JENKINS-29527) Fixed bug in inheritance blocking.
* [JENKINS-31860](https://issues.jenkins-ci.org/browse/JENKINS-31860) `ClassCastException` when used with multibranch projects.

### Version 1.2 (Apr 19, 2014)

* Allow a job to not inherit from global ACL ([JENKINS-10593](https://issues.jenkins-ci.org/browse/JENKINS-10593))

### Version 1.1 (Nov 11, 2013)

* Using an extension point in Jenkins 1.535.
* Better diagnosis for a form-related error.

### Version 1.0, 1.0.1, 1.0.2 (Oct 04, 2013)

* Split from Jenkins core.
