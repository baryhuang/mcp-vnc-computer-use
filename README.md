# MCP Server - Vnc MacOs Use
**The only MCP (Model Context Protocol) server that allow LLM to remote control any MacOS machines natively.**

Support Any MacOS Version (If apple documentation doesn't lie)
Apple authentication is hectic, you know them...you only other option is commercial closed-source RealVnc!

**Literally FREE because you can use your Claude plan with this MCP. No extra API charges are required!**

**No setup required on MacOs except for enabling sharing.**

## Why Critical
- You can use any LLM with this MCP server. You don't have to stick to Claude computer-use.
- No setup required on MacOs machines at all! As long as Screen Sharing is enabled, you can control any Mac machine using LLM. All current computer-use variants (including OOTB) requires running a python app in the background. Hectic!
- Use great Claude Desktop as UI! (You typically found a just-okay python UI in other projects)

** Limitation
The results highly depending on how the driving model return the accurate screen coordinates. For now I need the Anthropic api key to use "computer-use-2024-10-22".
If you are using other MCP Clients, you can skip this call and allow your driving model to do their things. Such as, gpt-4o may work just fine.


## Features
- Support for Apple Authentication (protocol 30) only for now
- Compatible with macOS Screen Sharing

## Prerequisites
- Python 3.11 or higher
- Network access to VNC server

## Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/mcp-vnc-macos-use.git
cd mcp-vnc-macos-use

# Install dependencies
pip install -e .
```

### Dependencies

This package requires the following dependencies:
- Python 3.10 or higher
- MCP (Model Context Protocol) >=1.4.1
- python-dotenv >=1.0.1
- Pillow >=10.0.0
- pyDes >=2.0.1
- cryptography >=44.0.0
- anthropic >=0.15.0 (for the vnc_macos_plan_screen_actions tool)

**Note:** The Anthropic package is a required dependency for this MCP server. It is used by the vnc_macos_plan_screen_actions tool to interact with Claude's computer use capability.

## Docker

### Building the Docker Image

```bash
# Build the Docker image
docker build -t mcp-vnc-macos-use .
```

## Usage with Claude Desktop

### Docker Usage

You can configure Claude Desktop to use the Docker image by adding the following to your Claude configuration:

```json
{
  "mcpServers": {
    "vnc-macos-use": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "buryhuang/mcp-vnc-macos-use:latest"
      ]
    }
  }
}
```

## Cross-Platform Publishing

To publish the Docker image for multiple platforms, you can use the `docker buildx` command. Follow these steps:

1. **Create a new builder instance** (if you haven't already):
   ```bash
   docker buildx create --use
   ```

2. **Build and push the image for multiple platforms**:
   ```bash
   docker buildx build --platform linux/amd64,linux/arm64,linux/arm/v7 -t buryhuang/mcp-vnc-macos-use:latest --push .
   ```

3. **Verify the image is available for the specified platforms**:
   ```bash
   docker buildx imagetools inspect buryhuang/mcp-vnc-macos-use:latest
   ```

## Usage

The server provides VNC functionality through MCP tools.

### Starting the Server

```bash
mcp_server_vnc_macos_use
```

### Using the Tools

#### Capturing a Screenshot with vnc_get_screen

```json
{
  "host": "vnc-server-hostname-or-ip",
  "port": 5900,
  "password": "your-vnc-password",
  "username": "your-username"
}
```

#### Using get_actions with Claude's Computer Use

The `get_actions` tool allows you to leverage Anthropic's Claude models with computer use capability to generate actions based on prompts. This tool requires an Anthropic API key and a screenshot image in base64 format.

```json
{
  "prompt": "Open Firefox and go to google.com",
  "anthropic_api_key": "YOUR_ANTHROPIC_API_KEY",
  "image_base64": "required_base64_encoded_screenshot"
}
```

**Note:** The `image_base64` parameter is required - Claude needs visual context to interact with the computer interface. You can typically get this by first calling the `vnc_macos_get_screen` tool, which returns a base64-encoded screenshot that can be used directly with the `get_actions` tool.

This tool uses Claude 3.5 Sonnet (claude-3-5-sonnet-20241022) and Anthropic's computer use functionality. The tool returns the generated actions from Claude with computer use capability. You can then use these actions to guide the VNC server interactions.

## Limitations

- **Authentication Support**: 
  - Only Apple Authentication (protocol 30) is supported
- Currently only supports capturing screenshots; does not support mouse or keyboard input

## Security Note

https://support.apple.com/guide/remote-desktop/encrypt-network-data-apdfe8e386b/mac
https://cafbit.com/post/apple_remote_desktop_quirks/

We only support protocol 30, which uses the Diffie-Hellman key agreement protocol with a 512-bit prime. This protocol is used by macOS 11 to macOS 12 when communicating with OS X 10.11 or earlier clients.

Here's the information converted to a markdown table:

| macOS version running Remote Desktop | macOS client version | Authentication | Control and Observe | Copy items or install package | All other tasks | Protocol Version |
|--------------------------------------|----------------------|----------------|---------------------|-------------------------------|----------------|----------------|
| macOS 13 | macOS 13 | 2048-bit RSA host keys | 2048-bit RSA host keys | 2048-bit RSA host keys to authenticate, then 128-bit AES | 2048-bit RSA host keys | 36 |
| macOS 13 | macOS 10.12 | Secure Remote Password (SRP) protocol for local only. Diffie-Hellman (DH) if bound to LDAP or macOS server is version 10.11 or earlier | SRP or DH,128-bit AES | SRP or DH to authenticate, then 128-bit AES | 2048-bit RSA host keys | 35 |
| macOS 11 to macOS 12 | macOS 10.12 to macOS 13 | Secure Remote Password (SRP) protocol for local only, Diffie-Hellman if bound to LDAP | SRP or DH 1024-bit, 128-bit AES | 2048-bit RSA host keys macOS 13 to macOS 10.13 | 2048-bit RSA host keys macOS 10.13 or later |  33 |
| macOS 11 to macOS 12 | OS X 10.11 or earlier | DH 1024-bit | DH 1024-bit, 128-bit AES | Diffie-Hellman Key agreement protocol with a 512-bit prime | Diffie-Hellman Key agreement protocol with a 512-bit prime |  30 |


Always use secure, authenticated connections when accessing remote VNC servers. This tool should only be used with servers you trust and have permission to access.

## License

See the LICENSE file for details. 
