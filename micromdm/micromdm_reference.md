# MicroMDM Implementation Details Reference

This document provides an overview of the MicroMDM architecture, components, and implementation details.

## System Architecture

MicroMDM is a Mobile Device Management (MDM) server implementation written in Go. It provides a framework for managing Apple devices through the Apple MDM protocol. The system is modular and designed to be extended.

### Core Components

- **Server**: The central component that initializes and coordinates all other components. Found in `server/server.go`.
- **MDM Module**: Handles the core MDM protocol communication. Located in the `mdm/` directory.
- **Command Platform**: Manages sending commands to devices and processing responses. Found in `platform/command/`.
- **Queue System**: Manages the queue of commands to be sent to devices. Implementations in `platform/queue/`.
- **Webhook System**: Forwards device events to external systems for processing. Implementation in `workflow/webhook/`.
- **API**: RESTful API for interacting with the MDM server. Various endpoints handled across different modules.

## Key Features

1. **Device Enrollment**: Supports both manual and DEP (Apple Business/School Manager) enrollment
2. **Command Scheduling**: API for sending MDM commands to devices
3. **Webhook Event Forwarding**: Real-time forwarding of device events to external systems
4. **API-Driven Architecture**: All functionality exposed through HTTP APIs
5. **Raw Command Support**: Ability to send raw plist commands for maximum flexibility

## Configuration Options

### Environment Variables

- `MICROMDM_SERVER_URL`: Public HTTPS URL of the server
- `MICROMDM_API_KEY`: API token for authentication
- `MICROMDM_WEBHOOK_URL`: URL to forward command responses to
- `MICROMDM_TLS`: Enable/disable TLS (boolean)
- `MICROMDM_TLS_CERT`: Path to TLS certificate
- `MICROMDM_TLS_KEY`: Path to TLS private key
- `MICROMDM_HTTP_ADDR`: HTTP listen address
- `MICROMDM_CONFIG_PATH`: Path to configuration directory

### Command Line Flags

The same options can be specified via command line flags:

```
micromdm serve -server-url=https://mdm.example.com -api-key=supersecret -command-webhook-url=https://webhook.example.com/endpoint
```

## API Endpoints

- `/v1/commands`: Schedule MDM commands
- `/v1/commands/{udid}`: Schedule raw commands or clear queue
- `/push/{udid}`: Send push notification to device
- `/mdm/enroll`: Enrollment endpoint
- `/mdm/checkin`: Device check-in endpoint
- `/mdm/connect`: Device command connections

## Event Types

### Checkin Events
- `mdm.Authenticate`: Device authentication during enrollment
- `mdm.TokenUpdate`: Device token update
- `mdm.CheckOut`: Device unenrollment

### Acknowledge Events
- `mdm.Connect`: Device responses to commands

## Tools

The repository includes API tools in the `tools/api/` directory with examples for all command types and operations.

## Deployment Considerations

1. **Security**: Requires HTTPS with valid certificates
2. **Network**: Devices need direct access to server and APNS
3. **Persistence**: Uses BoltDB for storage by default
4. **Webhooks**: External system needed to process device responses

## Example Usage

### Setting Up Environment
```bash
export MICROMDM_ENV_PATH="/path/to/.env"
```

### Sending Commands
```bash
./tools/api/send_push_notification DEVICE_UDID
./tools/api/commands/install_profile DEVICE_UDID /path/to/profile.mobileconfig
```

### Raw Commands
```bash
./tools/api/raw_command DEVICE_UDID /path/to/command.plist
```
