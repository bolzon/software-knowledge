## Enterprise Build Management with Multiple VOBs

### Understanding Your Large-Scale Architecture

With files split across multiple VOBs, your build process requires precise configuration management:

```
Multiple VOBs
├── VOB_A
│   ├── Project_Alpha (Chapter_1)
│   │   ├── Module_1
│   │   └── Module_2
│   └── Project_Beta (Chapter_2)
│       └── Module_3
└── VOB_B
    ├── Project_Gamma (Chapter_3)
    │   ├── Module_4
    │   └── Module_5
    └── Project_Delta (Chapter_4)
        └── Module_6
```

### Config Spec Management for Builds

**Config specs are your build recipes** - they determine exactly which versions of files from which VOBs get included in each build.

#### Build-Specific Config Specs
```bash
# View current config spec
cleartool catcs

# Set a specific config spec for a build
cleartool setcs -tag <view_name> <config_spec_file>

# Example config spec for a specific build
element * CHECKEDOUT
element * /main/integration_branch/LATEST
element /vob_a/project_alpha/... /main/release_2.1/LATEST
element /vob_b/project_gamma/module_4/... /main/hotfix_branch/LATEST
element * /main/LATEST
```

### Essential Build Commands

#### Multi-VOB Operations
```bash
# List all VOBs
cleartool lsvob

# Mount/unmount VOBs for build
cleartool mount <vob_tag>
cleartool umount <vob_tag>

# Check VOB health before builds
cleartool space <vob_tag>
cleartool describe vob:<vob_tag>
```

#### Build Configuration Management
```bash
# Create build-specific view
cleartool mkview -tag <build_view> -tmode <view_storage_path>

# Update view with latest config spec
cleartool setcs -current

# Force update entire view (for clean builds)
cleartool update -force -overwrite

# Update specific VOB areas only
cleartool update -force /vob_a/project_alpha/...
```

#### Version Selection for Builds
```bash
# Find specific versions across VOBs
cleartool find /vob_a /vob_b -version "brtype(release_branch)" -print

# List labels (build tags) across all VOBs
cleartool lstype -kind lbtype -invob <vob_tag>

# Apply build label to specific versions
cleartool mklabel -recurse <build_label> <path>

# Find files with specific label
cleartool find . -version "lbtype(<build_label>)" -print
```

#### Baseline and Release Management
```bash
# Create baseline labels for builds
cleartool mklbtype <baseline_name>@<vob>
cleartool mklabel -recurse <baseline_name> <project_path>

# Lock baselines to prevent changes
cleartool lock lbtype:<baseline_name>@<vob>

# Compare two baselines
cleartool diffbl -predecessor <baseline1> <baseline2>
```

### Build Integrator Workflows

#### Pre-Build Verification
```bash
# Check for hijacked files across all VOBs
cleartool ls -recurse -view_only | grep "hijacked"

# Verify no checkouts in integration areas
cleartool lsco -all -recurse <integration_path>

# Check config spec consistency
cleartool catcs | grep -E "(element|load)"
```

#### Build Execution
```bash
# Create clean build view with specific config
cleartool mkview -tag build_<timestamp> <storage_path>
cleartool setview build_<timestamp>
cleartool setcs <build_config_spec>

# Load only necessary areas (for performance)
cleartool update -add_loadrules <loadrule_file>

# Monitor build progress across VOBs
cleartool space -quick <vob_tag>  # Check VOB performance
```

#### Post-Build Operations
```bash
# Tag successful builds
cleartool mklabel -recurse BUILD_<version>_<date> .

# Create build manifest
cleartool ls -recurse -version | tee build_manifest.txt

# Archive config spec for build reproducibility
cleartool catcs > config_spec_build_<version>.txt
```

### Advanced Multi-VOB Build Commands

#### Cross-VOB Dependencies
```bash
# Find dependencies between VOBs
cleartool find . -name "*.h" -exec cleartool describe -short {} \;

# Check merge status across projects
cleartool findmerge <source_branch> <target_branch> -print

# Validate inter-project interfaces
cleartool diff -predecessor <interface_file>
```

#### Build Performance Optimization
```bash
# Use load rules to minimize view size
echo "load /vob_a/project_alpha/module_1" > loadrules.txt
echo "load /vob_b/project_gamma/module_4" >> loadrules.txt
cleartool setcs -current -add_loadrules loadrules.txt

# Parallel build support
cleartool update -workers <n> <path>

# Cache management for large builds
cleartool rgy -list  # Check region registry
```

#### Build Validation
```bash
# Verify build consistency across modules
cleartool describe -short <path>/... | sort | uniq -d

# Check for version skew between dependent modules
cleartool find . -name "version.h" -exec cleartool describe {} \;

# Validate no development branches in release build
cleartool find . -version "brtype(dev_*)" -print
```

### Integration Best Practices

1. **Config Spec Templates**: Maintain template config specs for different build types (debug, release, integration)

2. **Automated View Management**: Script view creation/destruction for builds to ensure consistency

3. **Build Labels**: Always label successful builds for traceability and rollback capability

4. **Load Rules**: Use selective loading to improve build performance with 100k+ files

5. **VOB Health Monitoring**: Regular VOB maintenance prevents build failures

6. **Parallel Processing**: Leverage ClearCase's ability to handle multiple VOB operations simultaneously

Remember: In enterprise environments with this scale, build reproducibility and configuration management are critical. Every build should be fully traceable through config specs and labels.
