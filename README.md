project_deploy
========

Deploy a project with Ansible

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
    project_deploy_strategy: "git" or "synchronize"

If you use the "git" strategy:
you must also set a repository

    project_git_repo: "git_repository"

you can set the git ref to deploy (can be a branch, tag or commit hash), defaults to "master"

    project_version: "master"

When using the synchonize method, we recommend using a .rsync-filter file in the source folder,
to exclude .git and other unneeded data to be transferred.

If you use the "synchronize" strategy:
you can set a timeout for the synchonize module

    project_deploy_synchronize_timeout: 30

you must set the path to your local source (default assumes the playbook in /ansible/)

    project_local_path: "../"


The source_path is used to fetch the tags from git, or synchronise via rsync. This way
you do not have to download/sync the entire project on every deploy

    project_source_path: "{{ project_root }}/source"

The build_path is used to build your project in, prior to moving it to the release folder.
If it exists, this path will be deleted before a new build is started
(under the assumption that it contains a failed previous build)

    project_build_path: "{{ project_root }}/build"

The project does not assume any package manager, you can enable whichever one(s) you use.
(only composer for the first release)

    project_has_composer: false

Default values to run composer install

    project_composer_binary: composer.phar
    project_command_for_composer_install: "{{ project_composer_binary }} install --no-ansi --no-dev --no-interaction --no-progress --optimize-autoloader --no-scripts"

All the files to copy to the remote system on deploy. These could contain config files.

    project_files: []

**Example:**

    project_files:
      - name: "some_file"           // <- optional, for your own reference and readability
        src: "local-path-to-file"   // <- relative or absolute, just like Ansible
        dest: "remote-path-to-file"
      - name: "some_other_file"
        src: "local-path-to-other-file"
        dest: "remote-path-to-other-file"

All the templates to copy to the remote system on deploy. These could contain config files.
Works the same as the project_files

    project_templates: []

The shared_children is a list of all files/folders in your project that need to be linked to someplace outside
the release. For example a logging directory or an uploads folder. These live in "/shared"

    project_shared_children: []

**Example:**

    project_shared_children:
      - path: "/app/sessions"
        src: "sessions"
      - path: "/web/uploads"
        src: "uploads"


The project_environment is a list of environment variables added to the various *_commands

    project_environment: {}

**Example:**

    project_environment:
      SYMFONY_ENV: "prod"


There are a few moments in this role where arbitrary command(s) can be run. These commands receive
the "project_environment" so deploys for different stages can be done by changing this.

    project_pre_build_commands: []
    project_post_build_commands: []

**Example:**

    project_post_build_commands:
      - "app/console cache:clear"
      - "app/console doctrine:migrations:migrate --no-interaction"
      - "app/console assets:install"
      - "app/console assetic:dump"

At the end of the role, the "current" symlink is set to the release. If you need to perform
your own actions before this happens, set "project_finalize" to false.

     project_finalize: true

When you're ready, perform the symlink task yourself (in post_tasks for example):

    - file: src={{ deploy.new_release_path }} dest={{ deploy.current_path }} state=link


Dependencies
------------

None.

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

You can also use the deploy module included with the role to cleanup old releases:

      post_tasks:
        - name: Remove old releases
          deploy: "path={{ project_root }} state=cleanup"

License
-------

LGPL

Author Information
------------------

Jasper N. Brouwer, jasper@nerdsweide.nl

Ramon de la Fuente, ramon@delafuente.nl
