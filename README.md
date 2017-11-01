project_deploy
========

Deploy a project with Ansible

### Changelog 4.0.0 BC BREAK. Set your galaxy version to v3.2.0 for Ansible 2.3 or less.

- replaced deprecated "include" with "include_tasks".

### Changelog 3.2.0

- added the variables "project_clean" and "project_keep_releases".

### Changelog 3.1.1

- Removed dependency on Ansible role `F500.project_deploy_module` (it is an Ansible Modules Extra module now)

### Changelog 3.1.0

- added option to set the release name explicitly. This allows for having the same release across all nodes.

### Changelog 3.0.0 BC BREAK. Set your galaxy version to v2.2.2 for Ansible 1.9 or less.

- Ansible V2 compatibility: Changed the hook mechanism. This is a breaking change with regards to Ansible v1.9, so until you upgrade to Ansible 2.0 you should lock the version of this role to "v2.2.2".

### Changelog 2.2.0

- added hooks. This feature allows for inclusion of arbitrary tasks at certain points in the process.

### Changelog 2.1.0

- added option to copy Composer, NPM and Bower folders from previous release

### Changelog 2.0.0  BC BREAK. Set your galaxy version to v1.0.0 for the old deploy module.

- remove the deploy module (it has been added to a separate module in f500.project_deploy_module)
- module renamed to deploy_helper

### Changelog 1.0.0

- added parameter unfinished_filename
- added fact deploy.unfinished_filename
- added state=query
- added project_unwanted_items
- added npm and bower package manager support
- added 'type' option to project_shared_children (file/directory)
- added a "unfinished_filename" to indicate a deploying state
- moved project_pre_build_commands after files/template copy
- changed state=cleanup to state=clean
- removed system folder and system_path variable
- removed "project_build_path"
- moved source folder to project_root/shared/source
- removed project_remove_git_folder (see project_unwanted_items)

Requirements
------------

Nothing beyond a remote server with ssh access and python. It *should* work on any
linux/unix OS although we've only tested it on Debian and Ubuntu.

Please let us know your experiences so we can update the platforms.

Role Variables
--------------
---

These variables must be set, they have no defaults:

    project_root: "path_to_project_on_the_target_machine"
    project_deploy_strategy: "git", "synchronize" or "s3"

This variable allows for setting your own release name instead of the generated timestamp.
When deploying to multiple nodes, setting this will keep the same release name between all nodes,
simplifying automated rollbacks (you can use the `set_fact` module to create unique name at runtime).

    project_release: "v3.12.6"

At certain key points in the role an option exists to include a task file of your own.
In order to use this option, create a tasks file and set the corresponding hook variable to its location:

    project_deploy_hook_on_initialize
    project_deploy_hook_on_update_source
    project_deploy_hook_on_create_build_dir
    project_deploy_hook_on_perform_build
    project_deploy_hook_on_make_shared_resources
    project_deploy_hook_on_finalize

**Example:**

    project_deploy_hook_on_perform_build: "{{ playbook_dir }}/deploy_hooks/perform-build.yml"

If you use the "git" strategy, you must also set a repository:

    project_git_repo: "git_repository"

you can set the git ref to deploy (can be a branch, tag or commit hash), defaults to "master":

    project_version: "master"

When using the synchonize method, we recommend using a .rsync-filter file in the source folder,
to exclude .git and other unneeded data to be transferred.

If you use the "synchronize" strategy, you can set a timeout for the synchonize module:

    project_deploy_synchronize_timeout: 30

and you can also tell synchronize to delete files that don't exist on source:

    project_deploy_synchronize_delete: true  # defaults to false

you must set the path to your local source (default assumes the playbook in /ansible/):

    project_local_path: "../"

If you use the "s3" strategy, you have to install python-boto on the target host and set AWS credentials

    aws_access_key: 'ACCESS_KEY'
    aws_secret_key: 'SECRET_KEY'

You also have to set s3 parameters, region, bucket, path and filename:

    project_s3_region: 'eu-west-1'
    project_s3_bucket: 'my-bucket'
    project_s3_path: '/my-app/'
    project_s3_filename: my-app-master.war

The source_path is used to fetch the tags from git, or synchronise via rsync.
This way you do not have to download/sync the entire project on every deploy:

    project_source_path: "{{ project_root }}/shared/source"

Files or folders to remove from the source when deploying:

    project_unwanted_items: [ '.git' ]


**DEPRECATED**
The build_path is used to build your project in, prior to moving it to the release folder.
If it exists, this path will be deleted before a new build is started
(under the assumption that it contains a failed previous build)

    project_build_path: "{{ project_root }}/build"

The project does not assume any package manager, you can enable whichever one(s) you use:

    project_has_composer: false
    project_has_npm: false
    project_has_bower: false

Default values to run composer install:

    project_composer_binary: composer.phar
    project_command_for_composer_install: "{{ project_composer_binary }} install --no-ansi --no-dev --no-interaction --no-progress --optimize-autoloader --no-scripts"

Default values to run npm install:

    project_npm_binary: npm
    project_command_for_npm_install: "{{ project_npm_binary }} install --production"

Default values to run bower install:

    project_bower_binary: bower
    project_command_for_bower_install: "{{ project_bower_binary }} install --production --config.interactive=false"

To speed up composer/npm/bower install it is possible to copy the vendor/node_modules/component directories from the previous release:

    project_copy_previous_composer_vendors: true
    project_copy_previous_npm_modules: true
    project_copy_previous_bower_components: true

You can also change the path of the installed vendors (relative to {{ deploy_helper.new_release_path }}):

    project_composer_vendor_path: vendor
    project_npm_modules_path: node_modules
    project_bower_components_path: components


The project_shared_path is used to set the path for your shared assets.
Required: No
Default: "{{ project_root }}/shared"

    project_shared_path: "/var/data/example"


All the files to copy to the remote system on deploy. These could contain config files:

    project_files: []

**Example:**

    project_files:
      - name: "some_file"           // optional, for your own reference and readability
        src: "local-path-to-file"   // relative or absolute, just like Ansible
        dest: "remote-path-to-file"
      - name: "some_other_file"
        src: "local-path-to-other-file"
        dest: "remote-path-to-other-file"

All the templates to copy to the remote system on deploy. These could contain config files.
Works the same as the project_files:

    project_templates: []

The shared_children is a list of all files/folders in your project that need to be linked to someplace outside
the release. For example a logging directory or an uploads folder. These live in "{{ project_shared_path }}":

    project_shared_children: []

**Example:**

    project_shared_children:
      - path: "/app/sessions"
        src: "sessions"
      - path: "/web/uploads"
        src: "uploads"


The project_environment is a list of environment variables added to the various *_commands:

    project_environment: {}

**Example:**

    project_environment:
      SYMFONY_ENV: "prod"


**DEPRECATED, PLEASE USE HOOKS**
There are a few moments in this role where arbitrary command(s) can be run. These commands receive
the "project_environment" so deploys for different stages can be done by changing this:

    project_pre_build_commands: []
    project_post_build_commands: []

**Example:**

    project_post_build_commands:
      - "app/console cache:clear"
      - "app/console doctrine:migrations:migrate --no-interaction"
      - "app/console assets:install"
      - "app/console assetic:dump"

At the end of the role, the "current" symlink is set to the release. If you need to perform
your own actions before this happens, set "project_finalize" to false:

     project_finalize: true

**Example (before v2.0.0):**

When you're ready, perform the symlink task yourself (in post_tasks for example):

    - file: src={{ deploy.new_release_path }} dest={{ deploy.current_path }} state=link

**Example (after v2.0.0):**

When you're ready, finalize the deploy with the module:

    - deploy_helper: path={{ project_root }} release={{ deploy_helper.new_release }} state=finalize

If you do want to finalize and have the role switch the "current" symlink, but
don't want to clean up old releases, set "project_clean" to false:

    project_clean: true

Or if you want to keep less or more than 5 releases, set "project_keep_releases"
to the desired number (minimum of 1):

    project_keep_releases: 5


Dependencies
------------

Before v2.0.0 : none
After v2.0.0 : f500.project_deploy_module
After v3.1.1 : none

Example Playbook
-------------------------

Because of the larger number of variables involved, we prefer to add them to the playbook as _vars_

Before you use this role, be sure to prepare the remote. If you want to deploy from a git repo, you need
the remote user to be able to clone the project (with SSH keys for example).

    - name: Deploy role example
      hosts: servers
      remote_user: deploy

      vars:
        project_root: /home/deploy/
        project_deploy_strategy: synchronize

      roles:
         - f500.project_deploy

You can also use the deploy module included with the role to clean up old releases
(this is done on finalize by default):

      post_tasks:
        - name: Remove old releases
          deploy_helper: "path={{ project_root }} state=clean"

License
-------

LGPL

Author Information
------------------

Jasper N. Brouwer, jasper@nerdsweide.nl

Ramon de la Fuente, ramon@delafuente.nl
