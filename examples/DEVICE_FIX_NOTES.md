# Device Variable Fix

## Issue
The `device` variable was undefined in `_load_pretrained_weights` function in visualize.py. This happened during refactoring when the function was split into smaller functions, creating an orphaned variable reference.

## Solution
1. **visualize.py**: Added `device` parameter to `load_pretrained_weights` function
   - Updated function signature: `load_pretrained_weights(model, pretrain_path, device, tensor_par_size=1, tensor_par_group=None)`
   - Updated function call in main() to pass the device parameter
   - Added device to docstring parameters

2. **intermediate_downscaling.py**: Fixed similar issues
   - Replaced duplicate `seed_everything` and `init_par_groups` with imports from utils.py
   - Fixed `self.target_transforms` to `target_transforms` (no self in function)
   - Changed device parameter to local_rank in `_load_pretrained_weights` call
   - Removed unused imports (random, numpy)

## Additional Improvements
- Created shared utils.py module to avoid code duplication
- Added comprehensive docstrings in English (no emojis)
- Added argparse to visualize.py for command-line parameters
- Applied Black formatting to all files

## Files Modified
1. `visualize.py` - Fixed device parameter issue, added argparse
2. `intermediate_downscaling.py` - Removed duplicate functions, fixed self reference
3. `utils.py` - New file with shared utility functions