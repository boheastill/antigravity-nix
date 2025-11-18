# Google Antigravity for NixOS

Nix flake for [Google Antigravity](https://antigravity.google) - Next-generation agentic IDE.

## Features

- **Auto-updating**: GitHub Actions workflow automatically checks for new versions daily
- **FHS Environment**: Uses buildFHSEnv for maximum compatibility
- **Declarative**: Integrate directly into your NixOS or Home Manager configuration
- **Overlay Support**: Easy integration with existing Nix setups

## Installation

### Option 1: Direct Installation with Nix Flakes

```bash
# Try it out without installing
nix run github:jacopone/antigravity-nix

# Install to your profile
nix profile install github:jacopone/antigravity-nix
```

### Option 2: NixOS Configuration

Add to your `flake.nix` inputs:

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    antigravity-nix = {
      url = "github:jacopone/antigravity-nix";
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };
}
```

Then add to your system packages:

```nix
# In your configuration.nix or modules
{ inputs, pkgs, ... }:
{
  environment.systemPackages = [
    inputs.antigravity-nix.packages.${pkgs.system}.default
  ];
}
```

### Option 3: Home Manager

```nix
# In your home.nix
{ inputs, pkgs, ... }:
{
  home.packages = [
    inputs.antigravity-nix.packages.${pkgs.system}.default
  ];
}
```

### Option 4: Using the Overlay

```nix
{
  nixpkgs.overlays = [
    inputs.antigravity-nix.overlays.default
  ];

  environment.systemPackages = with pkgs; [
    google-antigravity
  ];
}
```

## Usage

After installation, run:

```bash
antigravity
```

Or open a specific folder:

```bash
antigravity /path/to/project
```

## Requirements

- NixOS or Nix package manager
- Nix Flakes enabled
- `allowUnfree = true` in your Nix configuration (Antigravity is proprietary software)

### Enabling Unfree Packages

Add to your `configuration.nix`:

```nix
nixpkgs.config.allowUnfree = true;
```

Or for Nix flakes, add to your `flake.nix`:

```nix
nixpkgs = import inputs.nixpkgs {
  inherit system;
  config.allowUnfree = true;
};
```

## Development

### Building Locally

```bash
git clone https://github.com/jacopone/antigravity-nix.git
cd antigravity-nix
nix build
```

### Testing

```bash
nix run .#default
```

### Updating Version

Manual update:

```bash
./scripts/update-version.sh
```

The script will:
1. Check for the latest version from antigravity.google
2. Update version strings in `flake.nix` and `package.nix`
3. Fetch and update the SHA256 hash
4. Test the build
5. Commit changes if git is available

Automatic updates run daily via GitHub Actions and create pull requests.

## Project Structure

```
antigravity-nix/
├── flake.nix              # Main flake configuration
├── package.nix            # Package derivation with FHS environment
├── scripts/
│   └── update-version.sh  # Auto-update script
├── .github/
│   └── workflows/
│       └── update.yml     # GitHub Actions auto-update workflow
└── README.md
```

## How It Works

This package uses Nix's `buildFHSEnv` to create a Filesystem Hierarchy Standard environment for Antigravity. This approach:

- Provides all necessary system libraries
- Ensures binary compatibility
- Avoids manual ELF patching issues
- Maintains isolation from the rest of the system

The auto-update mechanism:
1. GitHub Actions runs daily (2 AM UTC)
2. Scrapes the antigravity.google download page for the latest version
3. Updates version strings and hashes
4. Tests the build
5. Creates a pull request with changes

## Troubleshooting

### Antigravity won't start

Check if unfree packages are enabled:

```bash
nix-instantiate --eval -E '(import <nixpkgs> {}).config.allowUnfree'
```

Should return `true`.

### Build fails with hash mismatch

The hash may have changed. Run the update script:

```bash
./scripts/update-version.sh
```

### Missing libraries error

The FHS environment should provide all necessary libraries. If you encounter missing library errors, please open an issue with:
- The error message
- Output of `antigravity --version` (if it runs at all)
- Your NixOS version: `nixos-version`

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Test your changes with `nix build`
4. Submit a pull request

## License

This Nix package is MIT licensed. Google Antigravity itself is proprietary software by Google.

## Related Projects

- [code-cursor-nix](https://github.com/jacopone/code-cursor-nix) - Cursor AI editor for NixOS (inspiration for this project)
- [nixpkgs](https://github.com/NixOS/nixpkgs) - Nix package collection

## Disclaimer

This is an unofficial package. Google Antigravity is a trademark of Google LLC.

## Acknowledgments

- Inspired by [code-cursor-nix](https://github.com/jacopone/code-cursor-nix)
- Thanks to the NixOS community for buildFHSEnv and flake patterns
