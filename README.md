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
    project_deploy_strategy: "git" or "rsync"

If you use the "git" strategy:
you must also set a repository

    project_git_repo: "git_repository"

you can set the git ref to deploy (can be a branch, tag or commit hash), defaults to "master"

    project_version: "master"

If you use the "rsync" strategy:
you must set the path to your local source

The default assumes your playbook is located in /ansible/

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
_Example:_
_project_files:_
_  - name: "some_file"_           // <- optional, for your own reference and readability
_    src: "local-path-to-file"_   // <- relative or absolute, just like Ansible
_    dest: "remote-path-to-file"_
_  - name: "some_other_file"_
_    src: "local-path-to-other-file"_
_    dest: "remote-path-to-other-file"_

    project_files: []

All the templates to copy to the remote system on deploy. These could contain config files.
Works the same as the project_files

    project_templates: []

The shared_children is a list of all files/folders in your project that need to be linked to someplace outside
the release. For example a logging directory or an uploads folder. These live in "/shared"
_Example:_
_project_shared_children:_
_  - path: "/app/sessions"_
_    src: "sessions"_
_  - path: "/web/uploads"_
_    src: "uploads"_

    project_shared_children: []

The project_environment is a list of environment variables added to the various *_commands
_Example:_
_project_environment:_
_  SYMFONY_ENV: "prod"_

    project_environment: ~

There are a few moments in this role where arbitrary command(s) can be run. These commands receive
the "project_environment" so deploys for different stages can be done by changing this .
_Example:_
_project_post_build_commands:_
_  - "app/console cache:clear"_
_  - "app/console doctrine:migrations:migrate --no-interaction"_
_  - "app/console assets:install"_
_  - "app/console assetic:dump"_

    project_pre_build_commands: []
    project_post_build_commands: []

At the end of the role, the "current" symlink is set to the release. If you need to perform
your own actions before this happens, set "project_finalize" to false, and when you're ready,
perform the symlink task yourself:
_- file: src={{ deploy.new_release_path }} dest={{ deploy.current_path }} state=link_

    project_finalize: true

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
        project_deploy_strategy: rsync

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
