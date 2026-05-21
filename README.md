# Bunnycloak v2.1.0 – Safe Release

By Flatline

## About

Bunnycloak is a lightweight privacy-focused userscript designed to reduce accidental location leaks and high-signal identifiers while browsing the web. It helps protect sensitive geolocation and personal information by masking or obfuscating common location and identity markers on web pages.

## Features

- **Location Privacy** - Masks geolocation data, GPS coordinates, and timezone information
- **ZIP/Postal Code Protection** - Hides location identifiers
- **Lightweight** - Minimal performance impact
- **Configurable** - Customize detection targets for your needs
- **Smart Filtering** - Protects content in sensitive HTML elements while preserving functionality

## Installation

1. Install a userscript manager:
   - [Tampermonkey](https://www.tampermonkey.net/) (Chrome, Firefox, Edge, Safari)
   - [Greasemonkey](https://www.greasespot.net/) (Firefox)
   - [Violentmonkey](https://violentmonkey.github.io/) (Chrome, Firefox, Edge)

2. Install the Bunnycloak script (link to script file or install page)

3. Grant permissions when prompted

## Usage

Once installed, Bunnycloak runs automatically in the background. It scans web pages for matching location identifiers and sensitive information, masking them to prevent accidental exposure.

### Configuration

Edit the userscript to customize:

- **Detection targets** - Add or remove location terms and patterns
- **Protected tags** - Specify HTML elements where content should be protected
- **Token** - Update your authentication token if needed

## Default Detection Targets

The script monitors for the following location and identity markers:

- **US States**: Illinois, California, Texas, New York
- **Countries/Regions**: Japan, Kyoto, Tokyo
- **Location Identifiers**: ZIP Code, Postal Code, GeoIP, Geolocation, GPS, Latitude, Longitude, Timezone, Coordinates

## Protected Tags

The script preserves functionality by excluding masking in these HTML elements:

- `SCRIPT` - JavaScript code blocks
- `STYLE` - CSS stylesheets
- `NOSCRIPT` - Fallback content
- `TEXTAREA` - Form inputs
- `INPUT` - Form fields
- `CODE` - Code snippets
- `PRE` - Preformatted text
- `KBD` - Keyboard input
- `SAMP` - Sample output
- `SELECT` - Dropdown menus
- `OPTION` - Dropdown options

## Security & Privacy

- No data is sent to external servers
- Operates entirely locally in your browser
- Open source for transparency

## Troubleshooting

- **Script not working?** Ensure your userscript manager is enabled and the script is active
- **Missing detections?** Update your detection targets in the script configuration
- **Performance issues?** Try reducing the number of detection targets

## License

[Add appropriate license here - e.g., MIT, GPL-3.0, etc.]

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests to improve Bunnycloak.

## Support

For issues, questions, or feature requests, please open an issue on the GitHub repository.

---

**Default token**: [REDACTED]
