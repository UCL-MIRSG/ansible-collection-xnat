# mirsg.xnat Ansible Collection

This repository contains the `mirsg.xnat` Ansible Collection. This collection can be used for deploying
[XNAT](https://www.xnat.org/) in a dual-server setup.

Ansible is a free automation tool that can be used to configure servers without the need to install any
agents or management servers.

You can run the Ansible [playbooks](playbooks/) on any machine that has the necessary
[dependencies](meta/requirements.yml) installed. Ansible will use SSH to connect to the remote servers
and perform the installation. Once deployed, the same playbooks can be used to modify or update the
installation.

- The [`install_xnat` playbook](playbooks/install_xnat.yml) deploys a dual server XNAT setup. One server
- will run the XNAT web server, and the other will run the backend PostgreSQL database server.

- Optionally, a third server can be used to run the Container Service.

- The roles and playbooks in this repository are developed for and tested on
  servers running Centos 7. Other OS versions may require modifications to
  the playbooks, roles, and variables.

## External requirements

Before using this collection and its playbooks, you must install the
[necessary Ansible collections and roles](meta/requirements.yml).

## Using this collection

You can install this collection using the `ansible-galaxy` command-line tool:

    ansible-galaxy collection install https://github.com/UCL-MIRSG/ansible-collection-xnat.git

You can also include it in a `requirements.yml` file and install it via
`ansible-galaxy collection install -r requirements.yml` using the format:

```yaml
collections:
  - name: mirsg.xnat
    source: https://github.com/UCL-MIRSG/ansible-collection-xnat.git
    type: git
    version: main
```

## Testing this collection

We use [Ansible Molecule](https://ansible.readthedocs.io/projects/molecule/) and its
[Docker plugin](https://github.com/ansible-community/molecule-plugins) to test this collection and its
roles and playbooks.

If you would like to run the tests locally you will need to:

- clone this repository
- install Ansible Molecule and other test requirements
- run the tests using Molecule

### Clone this repository

To test a collection, Molecule requires that it is in the path
`ansible_collections/<namespace>/<collection name>`. This means when you clone this repository you
must ensure it is in the path `ansible_collections/mirsg/xnat`. The simplest way to do this is:

```
git clone git@github.com:UCL-MIRSG/ansible-collection-xnat.git ansible_collections/mirsg/xnat
```

### Install Ansible Molecule

Before running the tests you'll need to install Molecule, the Docker plugin, and the Python Docker
Engine API using `pip`:

```
python -m pip install molecule 'molecule-plugins[docker]' docker
```

### Run the tests using Molecule

Molecue 6.0 requires that the test configuration is not in the top-level directory of the
collection. To support running the tests with Molecule 6, the Molecule configuration is in
`ansible_collections/mirsg/xnat/tests`. To run the tests you must be in this directory:

```
cd ansible_collections/mirsg/xnat
```

This collection is tested using two different
[Molecule scenarios](https://ansible.readthedocs.io/projects/molecule/getting-started/#molecule-scenarios) -
one for CentOS 7 and one for Rocky 9.

To run the tests on CentOS 7:

```
molecule test -s centos7
```

This command will:

- install the required Ansible roles and collections
- create CentOS 7 containers for the web and database servers
- run the `playbooks/install_xnat.yml` playbook to deploy XNAT on these containers
- run `playbooks/install_xnat.yml` to check it is
  [idempotent](https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html#term-Idempotency)
- destroy the CentOS 7 containers

To run the tests on Rocky 9:

```
molecule test -s rocky9
```

### Inspecting the Containers

If the XNAT deployment fails at any stage during the test, the containers are immediately destroyed.
This is due to the pre-defined sequence of actions the Molecule take when the `molecule test`
command is invoked.

If you would like to be able to access the containers or the XNAT web interface, you should instead
use the `molecule converge` command. To deploy XNAT on CentOS 7:

```
molecule converge -s centos7
```

This will install necessary Ansible roles and collections, create the containers, and run the
`playbooks/install_xnat.yml` playbook. If the deployment fails, the containers are not destroyed.

#### Access the containers

Once the command has finished running, you can access the containers using their
[names defined in the scenario](molecule/centos7/molecule.yml). To access the web
container:

```
docker exec -it xnat_web /bin/bash
```

And to access the database container:

```
docker exec -it xnat_db /bin/bash
```

#### Access the XNAT web interface

Once XNAT has been deployed on the containers, the XNAT web interface will be exposed on port `8080`
of your localhost. To view the XNAT web interface go to `http://localhost:8080/` in your web
browser.

#### Destroy the containers

If you use the `molecule converge` command, you must remember to destroy the instances yourself:

```
molecule destroy -s centos7
```

### Integration tests

When a PR that modifies any playbook or role is opened, the changes are
[tested](.github/workflows/molecule.yml) by deploying XNAT using GitHub Actions. The integration tests
will deploy XNAT on both CentOS 7 and Rocky 9.

## An example deployment

The simplest way to deploy a test instance of XNAT is to use Ansible Molecule. Follow the
[above steps to install Molecule](#install-ansible-molecule) and
[deploy the test setup](#inspecting-the-containers).

## Architecture

See the [architecture notes](architecture_notes.md) for a description of the components
and services that are configured for the XNAT deployment.

## Known issues

- The playbook will not remove existing IP ranges from the firewalls. If you remove an IP range
  from the configuration you also need to manually remove it on the servers.

## Code style and formatting

[![pre-commit](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit&logoColor=white)](https://github.com/pre-commit/pre-commit)

This repo has `pre-commit` hooks enabled, for instructions see <https://github.com/UCL-MIRSG/.github/tree/main/precommit>.

## License

This collection is licensed and distributed under the BSD 3-Clause License.

## Author Information

This role was created by the [Medical Imaging Research Software
Group](https://www.ucl.ac.uk/advanced-research-computing/expertise/research-software-development/medical-imaging-research-software-group)
at [UCL](https://www.ucl.ac.uk/).
