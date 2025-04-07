# Prompt Editor App

A sophisticated Markdown-based prompt editor for interacting with AI models. This application allows users to create, format, and manage prompts with dedicated formatting options for AI interactions. The editor includes advanced versioning, multi-language support, and AI-assisted prompt generation capabilities.

## Features

### Core Editing Features
- **Markdown Output**: Creates clean Markdown output that can be sent directly to AI models
- **Rich Text Editing**: Format text with bold, italic, and lists
- **Title Formatting**: Format any line as a title (H6) with a single click
- **Toggle Format**: Click on a formatted title to revert it to normal text
- **Line-by-Line Control**: Each line maintains its own independent formatting
- **Live Preview**: See how your formatted prompt will appear
- **Tab Indentation**: Support for tab indentation in lists
- **Text Coloring**: Apply different colors to your text for visual organization

### Advanced Features
- **Multi-language Support**: Create and manage prompt versions in multiple languages:
  - 🇧🇷 Portuguese (BR)
  - 🇵🇹 Portuguese (PT)
  - 🇺🇸 English (US)
  - 🇪🇸 Spanish (ES)
- **Versioning System**: Save up to 5 different versions of your prompt
  - Name and rename versions
  - Switch between versions easily
  - Create different language variants for each version
  - View version history with timestamps
- **AI-Assisted Prompt Creation**: Get AI suggestions to help write or refine your prompts and also create from pre defined templates
- **Copy/Paste Support**: Special handling for Markdown formatting in copy and paste operations
- **Export Options**: Export your prompt version data as JSON

## Usage

The editor features a comprehensive toolbar with the following options:

### Formatting Tools
- **Title Button (T)**: Format the current line as a title
- **Bullet List**: Create an unordered list
- **Bold**: Format selected text as bold
- **Text Color**: Apply different colors to your text
- **AI Assistant**: Toggle AI mode to get help creating or improving your prompt

### Version Management
- **Language Selector**: Choose which language to edit your prompt in
- **Version Selector**: Navigate between different saved versions
- **Save Button**: Save your changes to the current version
- **New Prompt**: Create a new prompt (discards unsaved changes)

## Main Views
- **Editor Panel**: Rich text editor with formatting tools
- **Output Panel**: 
  - Markdown Output: See the Markdown that will be sent to AI models
  - Versioning View: Examine the structure of saved versions and language variants

## Technical Details

This application is built with:

- **Vue 3**: For the reactive UI
- **Quasar Framework**: For UI components and styling
- **TypeScript**: For type safety and better developer experience
- **Turndown**: For converting HTML to Markdown

The editor uses a contenteditable div with proper Markdown conversion to ensure reliable output formatting.

## Installation

```bash
# Install dependencies
yarn install
# or
npm install

# Start the development server
quasar dev

# Build for production
quasar build
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

[MIT](LICENSE)
