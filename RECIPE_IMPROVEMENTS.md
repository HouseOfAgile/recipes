# HoA Companion Recipe v1.0 Improvements

## Overview

This PR introduces significant improvements to the hoa-companion Symfony Flex recipe, moving from v0.5 to v1.0 with a focus on modern vendor-based integration and clean environment management.

## Key Changes

### ❌ **What v0.5 Did (Problems):**
- Created local copies of `docker/` and `scripts/` directories
- Added composer scripts for copying files from vendor
- Empty environment variable management (`env: {}`)
- Required manual maintenance of copied files

### ✅ **What v1.0 Does (Solutions):**
- **Vendor-based Integration**: No local file copying, uses `vendor/houseofagile/hoa-companion/` paths directly
- **Environment File Management**: Creates `.env.dev` and `.env.prod` with proper hoa-companion blocks
- **Clean Makefile**: Provides complete Makefile that uses vendor paths and runs composer inside app container
- **No Composer Scripts**: No need for copy commands in project composer.json
- **Update-Safe**: When package updates, everything stays in sync automatically

## Recipe Structure

### v1.0 Recipe Features:
```json
{
    "copy-from-recipe": {
        "Makefile": "Makefile",
        ".env.dev": ".env.dev",
        ".env.prod": ".env.prod"
    },
    "gitignore": [
        ".env.dev",
        ".env.prod", 
        ".env.staging"
    ]
}
```

### Template Files Created:
- **Makefile**: Complete Makefile with vendor-based paths and composer commands in app container
- **.env.dev**: Development environment with hoa-companion blocks and PostgreSQL/Redis configuration
- **.env.prod**: Production environment with production-specific settings

## Benefits

### For Users:
1. **Simple Installation**: `composer require houseofagile/hoa-companion` - recipe handles everything
2. **No Manual Steps**: No copy scripts needed in composer.json
3. **Clean Repository**: Only necessary files in project root
4. **Update-Safe**: `composer update` keeps everything synchronized
5. **Modern Architecture**: Follows Symfony Flex best practices

### For Maintainers:
1. **Reduced Support**: No issues with outdated copied files
2. **Consistent Behavior**: All projects use same vendor-based approach
3. **Version Control**: Easy to track what version of hoa-companion tools users have

## Environment Variable Management

The new recipe creates clean environment files with proper block structure:

```bash
###> houseofagile/hoa-companion ###
TAG=0.1.0
APP_NAME=myproject
# ... all hoa-companion variables ...
###< houseofagile/hoa-companion ###
```

This follows Symfony Flex standards for environment variable management.

## Makefile Improvements

The provided Makefile includes:
- **Vendor Path Detection**: Automatically uses correct vendor paths
- **Stack Detection**: Detects frankenphp vs standard Docker stack
- **Composer Integration**: Runs composer commands inside app container
- **Environment Flexibility**: Supports dev/staging/prod environments
- **Complete Feature Set**: All common Docker operations included

## Migration from v0.5

Projects using v0.5 can migrate by:
1. Removing local `docker/` and `scripts/` directories
2. Removing copy scripts from composer.json
3. Running `composer update houseofagile/hoa-companion` 
4. The recipe will automatically create the new structure

## Testing

This recipe has been tested with:
- ✅ Fresh Symfony projects
- ✅ FrankenPHP Docker stack
- ✅ PostgreSQL and Redis services  
- ✅ Composer commands in app container
- ✅ Environment file loading
- ✅ Docker stack startup and operations

## Breaking Changes

⚠️ **Breaking Changes from v0.5:**
- No longer creates `docker/` and `scripts/` directories
- No longer adds composer copy scripts
- Changes from `dockerenv.*` to `.env.dev`/`.env.prod` structure
- Composer commands now run inside app container

## Backward Compatibility

While this is a breaking change, it provides a clear upgrade path and significantly improves the developer experience. Projects can continue using v0.5 until ready to upgrade.

## Implementation Notes

- Uses Symfony Flex `copy-from-recipe` for template files
- Follows semantic versioning (v0.5 → v1.0 due to breaking changes)
- Maintains same package name for seamless Composer integration
- Includes comprehensive post-install messaging for user guidance