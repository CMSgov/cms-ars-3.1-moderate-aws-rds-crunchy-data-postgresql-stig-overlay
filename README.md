# cms-ars-3.1-moderate-aws-rds-crunchy-data-postgresql-stig-overlay
**CMS’ ISPG (Information Security and Privacy Group) decided to discontinue funding the customization of MITRE’s Security Automation Framework (SAF) for CMS after September 2023. This repo is now in archive mode, but still accessible. For more information about SAF with current links, see https://security.cms.gov/learn/security-automation-framework-saf**

InSpec profile overlay to validate the secure configuration of AWS RDS hosted Crunchy Data PostgreSQL against [DISA](https://iase.disa.mil/stigs/)'s Crunchy Data PostgreSQL Security Technical Implementation Guide (STIG) Version 1, Release 1. (Applies to database versions 10, 11, 12 & 13) tailored for [CMS ARS 3.1](https://www.cms.gov/Research-Statistics-Data-and-Systems/CMS-Information-Technology/InformationSecurity/Info-Security-Library-Items/ARS-31-Publication.html) for CMS systems categorized as Moderate.

## Getting Started
### InSpec (CINC-auditor) setup
For maximum flexibility/accessibility, we’re moving to “cinc-auditor”, the open-source packaged binary version of Chef InSpec, compiled by the CINC (CINC Is Not Chef) project in coordination with Chef using Chef’s always-open-source InSpec source code. For more information: https://cinc.sh/

It is intended and recommended that CINC-auditor and this profile overlay be run from a __"runner"__ host (such as a DevOps orchestration server, an administrative management system, or a developer's workstation/laptop) against the target. This can be any Unix/Linux/MacOS or Windows runner host, with access to the Internet.

__For the best security of the runner, always install on the runner the _latest version_ of CINC-auditor.__ 

__The simplest way to install CINC-auditor is to use this command for a UNIX/Linux/MacOS runner platform:__
```
curl -L https://omnitruck.cinc.sh/install.sh | sudo bash -s -- -P cinc-auditor
```

__or this command for Windows runner platform (Powershell):__
```
. { iwr -useb https://omnitruck.cinc.sh/install.ps1 } | iex; install -project cinc-auditor
```
To confirm successful install of cinc-auditor:
```
cinc-auditor -v
```
> sample output:  _4.24.32_

Latest versions and other installation options are available at https://cinc.sh/start/auditor/.

### PSQL client setup

To run the PostgreSQL profile against an AWS RDS Instance, CINC-auditor expects the psql client to be readily available on the same runner system it is installed on.
 
For example, to install the psql client on a Linux runner host:
```
sudo yum install postgresql
```
To confirm successful install of psql:
```
which psql
```
> sample output:  _/usr/bin/psql_
```
psql –-version
```		
> sample output:  *psql (PostgreSQL) 12.9*

Test psql connectivity to your instance from your runner host:
```
psql -d postgresql://<master user>:<password>@<endpoint>.amazonaws.com/postgres
```		
> *sample output:*
> 
>  *psql (12.9)*
>  
>  *SSL connection (cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256)*
>  
>  *Type "help" for help.*
>  
>  *postgres-> \conninfo*
>  
>  *You are connected to database "postgres" as user "postgres" on host "(endpoint).us-east-1.rds.amazonaws.com" at port "5432".*
>  
>  *postgres=> \q*
>  
>  *$*

For installation of psql client on other operating systems for your runner host, visit https://www.postgresql.org/

## Inputs: Tailoring your scan to Your Environment

The following inputs must be configured in an inputs ".yml" file for the profile to run correctly for your specific environment. More information about InSpec inputs can be found in the [InSpec Profile Documentation](https://www.inspec.io/docs/reference/profiles/).

#### *Note* Windows and Linux InSpec Runner

There are current issues with how the profiles run when using a windows or linux runner. We have accounted for this in the profile with the `windows_runner` input - which we *default* to `false` assuming a Linux based InSpec runner.

If you are using a *Windows* based inspec installation, please set the `windows_runner` input to `true` either via your `inspec.yml` file or via the cli flag via, `--input windows_runner=true`

### Example Inputs You Can Use

```
# Changes checks depending on if using a Windows or Linux-based InSpec Runner (default value = false)
windows_runner: false


# These five inputs are used by any tests needing to query the database:
# Description: 'Postgres database admin user (e.g., 'postgres').'
pg_dba: 'postgres'

# Description: 'Postgres database admin password.'
pg_dba_password: ''

# Description: 'Postgres database hostname'
pg_host: ''

# Description: 'Postgres database name (e.g., 'postgres')'
pg_db: 'postgres'

# Description: 'Postgres database port (e.g., '5432')
pg_port: '5432'


# Description: 'Postgres users e.g., ["pg_signal_backend", "postgres", "rds_iam", "rds_pgaudit", "rds_replication", "rds_superuser", "rdsadmin", "rdsrepladmin"]'
pg_users: ["pg_signal_backend", "postgres", "rds_iam", "rds_pgaudit", "rds_replication", "rds_superuser", "rdsadmin", "rdsrepladmin"]

# Description: 'V-233592, V-233593 use this list of approved database extensions (e.g., ['plpgsql']).'
approved_ext: ["pgaudit"]

# Description: 'V-73011 uses this list of approved postgres-related packages (e.g., postgresql-server.x86_64, postgresql-odbc.x86_64)'
approved_packages: []

# Description: 'Postgres super users (e.g., ['postgres']).'
pg_superusers: []

# Description: 'Postgres normal user'
pg_user: ''

# Description: 'Postgres normal user password'
pg_user_password: ''

# Description: 'Postgres database table name'
pg_table: ''

# Description: 'User on remote database server'
login_user: ''

# Description: 'Database host ip'
login_host: ''

# Description: 'Database version'
# Change "12.x" to your version (This STIG applies to versions 10.x, 11.x, 12.x, and 13.x)
pg_version: '12.9'

# Description: 'Postgres ssl setting (e.g., 'on').'
pg_ssl: 'on'

# Description: 'Postgres audit log items (e.g., ['ddl','role','read','write']).'
pgaudit_log_items: ['ddl','role','read','write']

# Description: 'Postgres audit log line items (e.g. ['%m','%u','%c']).'
pgaudit_log_line_items: ['%m','%u','%c']

# Description: 'V-233520, V-233612 use this list of Postgres replicas from pg_hba.conf settings (e.g. ['127.0.0.1/32']).'
pg_replicas: []

# Description: 'Postgres max number of connections allowed (e.g., 100).'
pg_max_connections: 100

# Description: 'Postgres timezone (e.g., 'UTC').'
pg_timezone: 'UTC'
```
## Running This Overlay Directly from Github

```
# How to run
cinc-auditor exec https://github.com/CMSgov/cms-ars-3.1-moderate-aws-rds-crunchy-data-postgresql-stig-overlay/archive/master.tar.gz --input-file=<path_to_your_inputs_file/name_of_your_inputs_file.yml> --reporter json:<path_to_your_output_file/name_of_your_output_file.json>
```

### Different Run Options

  [Full exec options](https://docs.chef.io/inspec/cli/#options-3)

## Running This Overlay from a local Archive copy 

If your runner is not always expected to have direct access to GitHub, use the following steps to create an archive bundle of this overlay and all of its dependent tests:

(Git is required to clone the InSpec profile using the instructions below. Git can be downloaded from the [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) site.)

When the __"runner"__ host uses this profile overlay for the first time, follow these steps: 

```
mkdir profiles
cd profiles
git clone https://github.com/CMSgov/cms-ars-3.1-moderate-aws-rds-crunchy-data-postgresql-stig-overlay.git
cinc-auditor archive cms-ars-3.1-moderate-aws-rds-crunchy-data-postgresql-stig-overlay
cinc-auditor exec <name of generated archive> --input-file <path_to_your_input_file/name_of_your_input_file.yml> --reporter json:<path_to_your_output_file/name_of_your_output_file.json>
```

For every successive run, follow these steps to always have the latest version of this overlay and dependent profiles:

```
cd cms-ars-3.1-moderate-aws-rds-crunchy-data-postgresql-stig-overlay
git pull
cd ..
cinc-auditor archive cms-ars-3.1-moderate-aws-rds-crunchy-data-postgresql-stig-overlay --overwrite
cinc-auditor exec <name of generated archive> --input-file <path_to_your_input_file/name_of_your_input_file.yml> --reporter json:<path_to_your_output_file/name_of_your_output_file.json>
```

## Using Heimdall for Viewing the JSON Results

The JSON results output file can be loaded into __[heimdall-lite](https://heimdall-lite.mitre.org/)__ for a user-interactive, graphical view of the InSpec results. 

The JSON InSpec results file may also be loaded into a __[full heimdall server](https://github.com/mitre/heimdall2)__, allowing for additional functionality such as to store and compare multiple profile runs.

## Authors
* Eugene Aronne - [ejaronne](https://github.com/ejaronne)
* Danny Haynes - [djhaynes](https://github.com/djhaynes)

## Special Thanks
* Aaron Lippold - [aaronlippold](https://github.com/aaronlippold)
* Shivani Karikar - [karikarshivani](https://github.com/karikarshivani)

## Contributing and Getting Help
To report a bug or feature request, please open an [issue](https://github.com/CMSgov/cms-ars-3.1-moderate-aws-rds-crunchy-data-postgresql-stig-overlay/issues/new).

### NOTICE

© 2018-2022 The MITRE Corporation.

Approved for Public Release; Distribution Unlimited. Case Number 18-3678.

### NOTICE 

MITRE hereby grants express written permission to use, reproduce, distribute, modify, and otherwise leverage this software to the extent permitted by the licensed terms provided in the LICENSE.md file included with this project.

### NOTICE  

This software was produced for the U. S. Government under Contract Number HHSM-500-2012-00008I, and is subject to Federal Acquisition Regulation Clause 52.227-14, Rights in Data-General.  

No other use other than that granted to the U. S. Government, or to those acting on behalf of the U. S. Government under that Clause is authorized without the express written permission of The MITRE Corporation.

For further information, please contact The MITRE Corporation, Contracts Management Office, 7515 Colshire Drive, McLean, VA  22102-7539, (703) 983-6000.

### NOTICE 

DISA STIGs are published by DISA IASE, see: https://iase.disa.mil/Pages/privacy_policy.aspx
