# Visualization Script Updates

## Added Features

### 1. Comprehensive Documentation
- Added module-level docstring explaining the script's purpose and usage
- Added detailed docstrings for all functions with proper Args, Returns, and Notes sections
- Added inline comments throughout the code explaining key operations

### 2. Command Line Arguments (argparse)
All hardcoded parameters can now be overridden via command line:

```bash
python visualize.py config.yaml [options]

Options:
  --index INDEX          Index of test sample to visualize (default: 0)
  --variable VARIABLE    Variable to visualize (default: total_precipitation_24hr)
  --master-port PORT     Master port for distributed training (default: 29500)
  --data-type TYPE       Override data type from config (choices: float32, bfloat16)
```

Example usage:
```bash
# Visualize precipitation for the first sample
python visualize.py ../configs/interm_8m_ft.yaml

# Visualize temperature for the 10th sample
python visualize.py ../configs/interm_8m_ft.yaml --index 10 --variable 2m_temperature_max

# Use different master port for distributed training
python visualize.py ../configs/interm_8m_ft.yaml --master-port 29501
```

### 3. Improved Code Structure
- All hardcoded values have defaults but can be overridden
- Better separation of concerns with clear comments
- Automatic data key selection based on config
- More descriptive variable names and comments

## Key Changes Made

1. **Documentation**:
   - Module docstring with usage examples
   - Function docstrings following Google/NumPy style
   - Inline comments explaining complex operations
   - No emojis, all comments in English

2. **Command Line Interface**:
   - `--index`: Choose which test sample to visualize
   - `--variable`: Select which climate variable to visualize
   - `--master-port`: Configure distributed training port
   - `--data-type`: Override precision setting from config

3. **Code Quality**:
   - Applied Black formatter for consistent style
   - Created shared utils.py module
   - Added comprehensive error handling
   - Improved readability with better comments

## Backward Compatibility
All changes maintain backward compatibility. The script can still be run with just:
```bash
python visualize.py config.yaml
```
And it will use all the default values (same as the previous hardcoded values).