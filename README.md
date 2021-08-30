[Ondrej Sika (sika.io)](https://sika.io) | <ondrej@sika.io>

# Gitlab Management Training

    Ondrej Sika <ondrej@ondrejsika.com>
    https://github.com/ondrejsika/gitlab-management-training

### Any Questions?

Write me mail to <ondrej@sika.io>

## Course

## About Me - Ondrej Sika

**Freelance DevOps Engineer, Consultant & Lecturer**

- Complete DevOps Pipeline
- Open Source / Linux Stack
- Cloud & On-Premise
- Technologies: Git, Gitlab, Gitlab CI, Docker, Kubernetes, Terraform, Prometheus, ELK / EFK, Rancher, Proxmox

## Star, Create Issues, Fork, and Contribute

Feel free to star this repository or fork it.

If you found bug, create issue or pull request.

Also feel free to propose improvements by creating issues.

## Live Chat

For sharing links & "secrets".

<https://tlk.io/sika-gitlab>

## Agenda

- What is Gitlab
- Installation
- Configuration
- Backup / Restore
- User Management
- Gitlab Projects
  - Repository
  - Issues
  - Wiki

## Git

Git is a version control system for tracking changes in computer files and coordinating work on those files among multiple people.

## Gitlab

Gitlab is (opensource) collaboration platform for software developers.

- Git repository hosting & management
- Issue Tracking
- Wiki
- CI / CD
- Monitoring
- Error Tracking

## Run VM for Gitlab (Terraform)

```terraform
module "gitlab" {
  source  = "ondrejsika/ondrejsika-do-droplet/module"
  version = "1.1.0"

  count = 1

  image = "debian-10-x64"
  size  = "s-8vcpu-16gb"
  tf_ssh_keys = [
    data.digitalocean_ssh_key.ondrejsika.id
  ]
  zone_id      = local.sikademo_com_zone_id
  droplet_name = "gitlab${count.index}"
  record_name  = "gitlab${count.index}"
  user_data    = <<EOF
#cloud-config
ssh_pwauth: yes
password: asdfasdf
chpasswd:
  expire: false
EOF
}
```

## Install Gitlab

- https://about.gitlab.com/install/
- https://about.gitlab.com/install/?version=ce
- https://about.gitlab.com/install/ce-or-ee/

```
export LC_ALL="en_US.UTF-8"
export LC_CTYPE="en_US.UTF-8"
```

```
sudo apt-get update
sudo apt-get install -y curl openssh-server ca-certificates perl
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash
sudo EXTERNAL_URL="https://gitlab0.sikademo.com" apt-get install gitlab-ee
```

### Get `root` password

```
cat /etc/gitlab/initial_root_password
```

## Gitlab Backup & Restore

Docs: https://docs.gitlab.com/ee/raketasks/backup_restore.html

## Backup Gitlab

Docs: https://docs.gitlab.com/ee/raketasks/backup_restore.html#back-up-gitlab

```
sudo gitlab-backup create
```

Backup is by default in `/var/opt/gitlab/backups`

```terminal
root@gitlab0:~# ls -la /var/opt/gitlab/backups
total 320
drwx------  2 git  root   4096 Aug 30 05:17 .
drwxr-xr-x 23 root root   4096 Aug 30 05:16 ..
-rw-------  1 git  git  317440 Aug 30 05:17 1630300669_2021_08_30_14.2.1-ee_gitlab_backup.tar
```

## Restore Gitlab

Docs: https://docs.gitlab.com/ee/raketasks/backup_restore.html#restore-gitlab

Stop the processes that are connected to the database. Leave the rest of GitLab running:

```
gitlab-ctl stop puma
gitlab-ctl stop sidekiq
# Verify
gitlab-ctl status
```

Run restore

```
gitlab-backup restore BACKUP=1630300669_2021_08_30_14.2.1-ee
```

Reconfigure, restart and check GitLab:

```
gitlab-ctl reconfigure
gitlab-ctl restart
gitlab-rake gitlab:check SANITIZE=true
gitlab-rake gitlab:doctor:secrets
```

## Upgrade Gitlab

Docs: https://docs.gitlab.com/ee/update/#linux-packages-omnibus-gitlab

```
apt update
apt upgrade
```

## Configuration

All configuration is in `/etc/gitlab/gitlab.rb`.

```
vim /etc/gitlab/gitlab.rb
```

After configuration change, you have to run `gitlab-ctl reconfigure`.

```
gitlab-ctl reconfigure
```

### SMTP

Docs: https://docs.gitlab.com/omnibus/settings/smtp.html

```rb
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = 'maildev0.sikademo.com'
gitlab_rails['smtp_port'] = 25
gitlab_rails['smtp_domain'] = 'maildev0.sikademo.com'
gitlab_rails['smtp_tls'] = false
gitlab_rails['smtp_openssl_verify_mode'] = 'none'
gitlab_rails['smtp_enable_starttls_auto'] = false
gitlab_rails['smtp_ssl'] = false
gitlab_rails['smtp_force_ssl'] = false
```

```
gitlab-ctl reconfigure
```

See all emails in Maildev: http://maildev0.sikademo.com/#/

### S3 Backup Config

```ruby
gitlab_rails['backup_upload_connection'] = {
  'provider' => 'AWS',
  'aws_access_key_id' => 'admin',
  'aws_secret_access_key' => 'asdfasdf',
  'endpoint'              => 'http://minio0.sikademo.com:9000',
  'path_style' => true
}
gitlab_rails['backup_upload_remote_directory'] = 'gitlab-backups'
```

```
gitlab-ctl reconfigure
gitlab-backup create
```

See backup in Minio: http://minio0.sikademo.com, user: `admin`, password `asdfasdf`

## Settings & Management

### Global Gitlab Settings

- Users
- Groups
- Projects
- Jobs (CI)
- Runners (CI)
- Gitaly servers (git operations)

### Management

- Users
- Groups
- Projects

### Groups

- Create / Edit / Delete
- Visibility level
- Subgroups
- Group settings

### Users

- Edit user settings
- Impersonation
- Keys
- Identities (LDAP, ...)

### Project Components

- Repository
- Issues
- Merge Requests
- Wiki
- Registry
- CI / CD

### Git Repository

- Git Repository (SSH & HTTPS access)
- History
- Branches / Tags
- Graph (of Git tree)
- Contributors, Charts

### Web IDE

- Simple IDE in browser
- Great for Docs, CI scripts
- Hot fixes (create new branch with merge request on change)
- Stage
- Multiple changed files in single commit

### Issues

- Project / Group view
- List / Kanban boards view
- Labels
- Milestones

#### Labels

- Filters
- Trackers - Bug, Feature, ...
- Priority - Low, Medium, High, ...
- Stages (for Kanban boards) - Todo, Doing, Review, ...

#### Kanban Boards

- Project / Group Kanban board
- Uses labels as stages
- Great for agile & sprints

### Merge Requests

- Request for review & merge of branch
- Created from branch or issue
- Discussion
- Approvals
- CI / CD before merge

#### Setup merging strategy

Project Setting -> General -> Merge Request

### Merge methods (strategies)

- Merge commit (default)
- Merge commit with semi-linear history
- Fast-forward merge (I recommend, requires rebase)

### Docker Registry

- Place to store your Docker images
- Same access policy as project & git repository
- Required configuration of `gitlab_registry_url` in gitlab.rb

### Wiki

- Supports Markdown
- Simple project related wiki
- Git versioned / runs locally using tool Gollum

### Snippets

- Tool for sharing code snippets
- Supports multiple files
- Syntax Highlight
- Discussion

## Workflow

## Merge vs Rebase merge strategy

### Merges (Default)

- `+` No history rewrites
- `-` Awful & non intuitive history

### Rebase (I recommend & use)

- `+` Single line history - easy to read and understand
- `-` History rewrites
- `-` Force pushes (in some cases)

## Gitlab Workflow - Feature, Bug

- Start issue & label it to feature (for features)
- Start issue & assign developer & label it to bug (for bugs)
- When task is ready, PM (project manager) moves it into TODO
- Developer pick ticket and start WIP merge request with branch
- Fetch repository and make some changes or work in Web IDE
- Pushes commits to Gitlab
- Developer remove WIP from Merge Request
- Someone does code review & CI / CD pipeline runs tests
- Someone can approve the Merge Request
- Maintainer (typically who has access to master) merge it
- After merge, issue will be closed and everything is done

## Gitlab Workflow - Hotfix

- Typically quick fix using Web IDE
- Select commit to new branch & create Merge Request in Web IDE
- Do review as fast you can
- Merge & Deploy

## Resources

### Terraform

- Terraform VM Module - https://github.com/ondrejsika/terraform-module-ondrejsika-do-droplet

### Examples

TODO

## Thank you! & Questions?

That's it. Do you have any questions? **Let's go for a beer!**

### Do you like the course? Tweet about it <3

Please, tweet something with `@ondrejsika`. Thanks :)

### Ondrej Sika

- email: <ondrej@sika.io>
- web: <https://sika.io>
- twitter: [@ondrejsika](https://twitter.com/ondrejsika)
- linkedin: [/in/ondrejsika/](https://linkedin.com/in/ondrejsika/)
- Newsletter, Slack, Facebook & Linkedin Groups: <https://join.sika.io>

### Next time?

Wanna to go for a beer or do some work together? Just [book me](https://book-me.sika.io) :)
