# Develocity Project-Level Access Control and User Account Creation

Develocity's "project-level access control" feature allows restricting certain data from certain projects to only be accessible by certain users. For Commonhaus projects, it is leveraged to prevent users and CI agents from publishing scans and remote build cache entries for projects other than their own. This helps to prevent a scenario where one project's credential is compromised and affects the entire Commonhaus ecosystem. The project-level access control feature is documented [here](https://docs.gradle.com/develocity/helm-admin/current/#project_level_access_control).

Onboarding a project to the Commonhaus Develocity instance requires creating a new Develocity project and project group. Users are associated with project group(s), which in turn contain one or more projects.

## Enabling Project-Level Access Control

Before creating the new project and project group, project-level access control must be enabled.

Navigate to `Administration > Access Control > Projects` and ensure that the "Enable project-level access control" box is checked and the "Allow data without an associated project" box is unchecked.

In addition to restricting users from publishing scans or remote cache entries for projects other than their own, enabling project-level access control has the effect of restricting build scan viewing and other anonymous permissions in the same way. To allow anonymous permissions to still apply, we can enable the "Anonymous permissions apply to all data" option. This can be done by navigating to `Administration > Advanced` and adding a new config parameter `feature.anonymousPermissionsApplyToAllData=ENABLED`.

Now that we have ensured project-level access control and the "Anonymous permissions apply to all data" option are enabled, we can create the project and project group corresponding to the project we are onboarding.

## Creating a New Develocity Project

1. Navigate to `Administration > Access Control > Projects`
2. Click "Add project"
3. Add, at a minimum, a project ID unique to the project being onboarded. This project ID will later be used when configuring the build with the Develocity Gradle Plugin or Maven Extension to associate the project to this project in Develocity.
    1. For now, don't worry about associating this project with any project groups - that will be done in the next step
4. Click "Save" to persist the new project

## Creating a New Develocity Project Group

1. Navigate to `Administration > Access Control > Projects`
2. Click "Add project group"
3. Add a project group ID unique to the project being onboarded. This project group will later be used to associate users with this project group.
4. Add the project you created to this project group
5. Click "Save" to persist the new project group

## User account creation

Now that the project and project group have been created, the necessary user accounts can be created. For developers to publish Build Scans and utilize the remote build cache, a Develocity user account with the `Developer` role is required. Similarly, a user account with the `CI Agent` role is required for CI builds. Both of these user accounts should be associated with the project group created in the previous step.
