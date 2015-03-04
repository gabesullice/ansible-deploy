Ansible Based Deployment
------------------------

### Usage
From the ansible directory, you will run:

    ansible-playbook <env>.yml # where env could be dev, stg, or prd

You may only wish to do portions of the provisioning process. To only execute a single stage of the process, you can use the `-t` flag:

    ansible-playbook <env>.yml -t setup # this would only create the directory structure

You may also exclude portions of the process with `--skip-tags`:

    ansible-playbook <env>.yml --skip-tags launch # Does everything but symlink ht(s)docs to app/current

### Advantages
- Start to finish
  - Creates vhosts, databases, and db users
  - Create local-settins.inc files per environment for you
- 2 commands to launch a site
  - `ansible-playbook <env>.yml`
  - `drush sql-sync @alias @alias`
- Declaritive syntax
  - Easy to reason about what is happening
  - Easy to customize
  - No more ruby or gem dependencies
  - YAML based, so in line w/ Drupal 8
- Idempotent
  - Run it as many times as you like
  - Run it to fix directory structure
  - Run it to fix permissions
- Fast
- Concise
- Gets us closer to being able to do across the board core updates
- Follows the same directory pattern as Cap
- Backwards compatible
  - Replace the cap directory w/ this, and set it up.
  - In theory, shouldn't break deployments going forward (needs testing)

### Disadvantages
- Different
- Need python, ansible installed

### Structure
The entry point begins in the top level YAML file. There should be one file per environment. Each file specifies roles to provision. This set-up has 3 roles: apache, mysql, and application. Within each roles, there may be 3 dirs: templates, tasks, and handlers.

- Roles
  - apache
    - Sets up the vhost, makes sure apache is reloaded whenever the vhost is updated
  - mysql
    - Creates the database, user, and sets the users passwod
  - application
    - Sets up the file structure
    - Pulls the repo
    - Creates a release
    - Sets up symlinks
    - Writes a local-settings.inc file
- Templates
  - Currently, there's vhost.j2 and local-settings.j2 (python's template extension)
  - vhost.j2 will write out a correct vhost file based on the variables in the top-level YAML file
  - local-settings.j2 will write out a local-settings file based on the variables in the top-level YAML file
- Tasks
  - Executed sequentially, starting with main.yml.
  - application/tasks/main.yml uses includes to break apart files, loosely based on duties
  - Some tasks may have a `notify` setting. This means that if that task creates a change on the system, it will notify a handler (see below) to be triggered when all tasks are done. For example, this is used by the apache role vhost task to reload apache if and when the vhost is altered, but not when the vhost remains the same.
- Handlers
  - Handlers are just like tasks, but they will not be run unless notified (see above) by a task.

### Tags
You may notice that many of the tasks define `tags`. These tags allow you to run specific parts of the site provisioning process independently. For example, if you want to provision a site, but not launch it on prod, you would run `ansible-playbook prd.yml --skip-tags launch`. Or, if a site was already launched and you wanted to deploy code changes, but not yet release it, you would run `ansible-playbook prd.yml -t deploy`.

##### Available tags
- setup
- deploy
- release
- launch
