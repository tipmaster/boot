# Perl Dependency Management

## Overview

CPAN modules installed **system-wide** to `/usr/local/lib/perl5/site_perl/5.42.0/`

Shared across all Perl projects (`/opt/lt`, `/opt/tax`, etc.) - no `use lib` needed.

## Installation

### 1. System packages (build dependencies)
```bash
sudo dnf install -y expat-devel libdb-devel MariaDB-devel gd-devel
```

### 2. Install CPAN modules
```bash
sudo cpanm -n --cpanfile /opt/lt/config/cpanfile --installdeps .
```

## Files

| File | Purpose |
|------|---------|
| `/opt/lt/config/cpanfile` | Authoritative module list (~200 modules) |

## Verify

```bash
perl -c /opt/lt/dispatcher.pl
```

Expected: `dispatcher.pl syntax OK` (with harmless BSON.pm/Any::Moose warnings)

## Troubleshooting

**Module not found:** Add to cpanfile, then:
```bash
sudo cpanm -n Module::Name
```

**Build failure:** Usually missing system library. Check error for `*.h` file, install corresponding `-devel` package.

| Module | Requires |
|--------|----------|
| XML::Parser | expat-devel |
| DB_File | libdb-devel |
| DBD::MariaDB | MariaDB-devel |
| GD | gd-devel |
