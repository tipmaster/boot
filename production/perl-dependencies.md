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

---

## Fedora 43+ Specific Issues

Fedora 43 uses GCC 15 with **C23 as the default standard**, causing build failures for older XS modules.

### Bit::Vector (blocks Date::Calc)

**Error:**
```
ToolBox.h:98:20: error: cannot use keyword 'false' as enumeration constant
```

**Cause:** C23 reserves `false`/`true` as keywords.

**Fix:** Force C17 standard:
```bash
sudo PERL_MM_OPT="OPTIMIZE=-std=gnu17" cpanm -n Bit::Vector
sudo cpanm -n Date::Calc
```

### Net::IDN::Encode (blocks Domain::PublicSuffix)

**Error:**
```
error: implicit declaration of function 'uvuni_to_utf8_flags'
```

**Cause:** Deprecated Perl API function (renamed to `uvchr_to_utf8_flags` in Perl 5.24+).

**Fix:** Patch and build manually:
```bash
cd /tmp
wget https://cpan.metacpan.org/authors/id/C/CF/CFAERBER/Net-IDN-Encode-2.500.tar.gz
tar xzf Net-IDN-Encode-2.500.tar.gz
cd Net-IDN-Encode-2.500
sed -i 's/uvuni_to_utf8_flags/uvchr_to_utf8_flags/g' lib/Net/IDN/Punycode.xs
perl Build.PL && ./Build && sudo ./Build install
sudo cpanm -n Domain::PublicSuffix
```

### Summary: Fedora vs AlmaLinux

| Issue | Fedora 43+ | AlmaLinux 9 |
|-------|------------|-------------|
| C23 `false`/`true` keywords | Fails (fix required) | Works |
| Net::IDN::Encode Perl API | Fails (patch required) | Fails (patch required) |
