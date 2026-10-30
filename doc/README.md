# Wi-Fi OnChip Silex Software Building Block

## Overview

This Software Building Block (SBB) demonstrates Wi-Fi OnChip Silex functionality on Renesas RA MCUs using the Renesas Silex UART Pmod. The reference application connects the MCU to a Wi-Fi access point, performs DNS lookup and ping operations, creates a TCP client socket, connects to a user-selected TCP server, and allows that server to control an on-board LED with `on` and `off` commands.

This document covers only:

- FPB-RA4E2
- FPB-RA6E2

## 1. Use Case Demonstrated

The SBB demonstrates a network-connected RA application that uses the Renesas Silex UART Pmod for Wi-Fi connectivity. The user can scan for Wi-Fi access points or manually enter access-point credentials. After connection, the application obtains the MCU IP address, performs DNS lookup and ping operations against a user-entered URL or IP address, then operates as a TCP client connected to a TCP server on the network.

After the TCP connection is established, the server can send:

- `on` to turn the on-board LED on.
- `off` to turn the on-board LED off.
- Any other text, which is treated as an invalid command and leaves the current LED state unaffected.

The MCU reports the LED state on the terminal and sends `Success` or `Invalid` acknowledgments back to the TCP server.

## 2. SBB Core Code

The SBB functionality is divided between Wi-Fi management and TCP/network operations.

### Wi-Fi services: `wifi_ep.c` and `wifi_ep.h`

The Wi-Fi module provides:

- `wifi_open()` to open and configure the Wi-Fi OnChip Silex module.
- `wifi_config()` to connect to the selected access point, verify the connection, and obtain the assigned MCU IP address.
- `wifi_scan()` to scan for available Wi-Fi access points.
- `wifi_set_ap_ssid()` to select the SSID and security type from a scan result.
- `socket_connected()` to check the Wi-Fi module connection status.
- `wifi_deinit()` to close the Wi-Fi module when connected.
- `str_to_lowercase()` to normalize received commands before command comparison.
- `set_led()` to control the designated user LED through the FSP I/O port interface.

The implementation maintains Wi-Fi state in `g_wifi` and stores up to `MAX_WIFI_SCAN_RESULTS` scan results in `g_scan_data`. The supplied configuration defines `MAX_WIFI_SCAN_RESULTS` as 10 and uses 32-byte Wi-Fi and socket buffers.

### TCP/network services: `tcp_socket.c` and `tcp_socket.h`

The TCP/network module provides:

- `dns_query()` for DNS resolution or IP-address validation.
- `ping()` for pinging a resolved address.
- `socket_create()` for creating an IPv4 TCP client socket.
- `tcp_connect()` for connecting the MCU TCP client to a server.
- `network_connectivity_check()` for checking the socket state.
- `tcp_send_data()` for sending data to the server.
- `tcp_recv_data()` for receiving server data.
- `socket_deinit()` for disconnecting when the socket is connected.

Socket state, server address, port, send buffer, receive buffer, and status are stored in `g_socket`.

## 3. Reusable SBB Code vs. Demo/Reference Application Code

### Reusable SBB-oriented code

The networking functionality is implemented in:

- `wifi_ep.c`
- `wifi_ep.h`
- `tcp_socket.c`
- `tcp_socket.h`

These modules provide the Wi-Fi, DNS, ping, TCP socket, data-transfer, connection-status, and LED-control operations used by the example.

They are not independent of the Renesas software and hardware environment. The implementation directly uses the Wi-Fi OnChip Silex APIs, FSP types, generated configuration, and FSP I/O port services.

### Demo/reference application code

The demonstration workflow is implemented in:

- `wifi_example_app.c`
- `wifi_example_app.h`

`wifi_example_app_run()` is the reference application's single entry point. It initializes the UART console, displays project information, collects access-point information, connects to Wi-Fi, runs the ping demonstration, creates and connects a TCP client socket, sends an initial `Hello`, and then waits for server commands.

The following behavior belongs to the reference application rather than the lower-level SBB services:

- Terminal menus and prompts
- Selection between Wi-Fi scanning and manual credential entry
- Display and selection of scan results
- User entry of SSID, security type, and password
- User entry of a ping target
- User entry of TCP server address and port
- Interpretation of `on` and `off` TCP messages
- Terminal status/error messages
- Sending `Success` and `Invalid` demonstration acknowledgments
- Breakpoint-based terminal error handling

## 4. Public Functions to Review First

For Wi-Fi integration, start with:

| Function | Purpose |
|---|---|
| `wifi_open()` | Opens the Wi-Fi OnChip Silex module. |
| `wifi_scan()` | Scans for available Wi-Fi access points. |
| `wifi_set_ap_ssid()` | Copies the selected scan result's SSID and security setting into the active Wi-Fi parameters. |
| `wifi_config()` | Connects to the configured access point and obtains the MCU IP address. |
| `socket_connected()` | Checks the Wi-Fi connection status. |
| `wifi_deinit()` | Closes the connected Wi-Fi module. |

For TCP and network integration, review:

| Function | Purpose |
|---|---|
| `dns_query()` | Resolves a URL or validates an IP address. |
| `ping()` | Performs the demonstrated ping operation. |
| `socket_create()` | Creates the IPv4 TCP client socket. |
| `tcp_connect()` | Connects the TCP client to the specified server and port. |
| `network_connectivity_check()` | Checks whether the socket remains connected. |
| `tcp_send_data()` | Sends a message through the socket. |
| `tcp_recv_data()` | Receives data through the socket. |
| `socket_deinit()` | Disconnects when the socket is connected. |

`wifi_example_app_run()` should be reviewed to understand how the reference application sequences these services.

## 5. Reference Board Configuration Assumptions

### Supported boards covered by this document

| Board | Silex UART Pmod connection | Additional documented requirement |
|---|---|---|
| FPB-RA4E2 | Connect the Renesas Silex UART Pmod to `PMOD1`, aligning pin 1 of the Pmod with pin 1 of PMOD1. | Close `E6` and open `E5` to use PMOD1 for UART. |
| FPB-RA6E2 | Connect the Renesas Silex UART Pmod to `PMOD1`, aligning pin 1 of the Pmod with pin 1 of PMOD1. | No additional board-specific requirement is stated in the supplied README. |

For both boards, the SBB source defines `USER_LED` as `BSP_IO_PORT_02_PIN_07`. `LED_ON` is `BSP_IO_LEVEL_HIGH` and `LED_OFF` is `BSP_IO_LEVEL_LOW` for these boards.

### Required hardware and network environment

The supplied README specifies:

- One supported Renesas RA board.
- One Renesas Silex UART Pmod.
- One Wi-Fi router/access point with internet access.
- Available Wi-Fi access-point credentials.
- One PC or laptop connected to the same Wi-Fi access point.
- A TCP server running on the PC and listening on a free port.
- A USB cable appropriate for the board for programming and debugging.

The README recommends a TCP server port greater than `10,000`.

### Software environment

The supplied README identifies:

- Renesas Flexible Software Package (FSP) `6.5.0`
- e2 studio `2026-04.2`
- LLVM Embedded Toolchain for Arm `21.1.1`
- Tera Term or a similar terminal console application

### Serial terminal configuration

The documented terminal settings are:

- Port: J-Link On-Board provided port
- Speed: `115200`
- Data: `8 bit`
- Parity: `none`
- Stop bits: `1 bit`
- Flow control: `none`
- Receive new-line: `CR`
- Transmit new-line: `CR`
- Local echo: enabled

## 6. What Must Be Changed When Porting

When integrating the SBB into another project or board, review these points.

### Silex UART Pmod connection

The reference hardware uses the Renesas Silex UART Pmod. For FPB-RA4E2 and FPB-RA6E2 it connects through PMOD1. FPB-RA4E2 additionally requires `E6` closed and `E5` open for PMOD1 UART operation.

If the hardware connection changes, the corresponding project peripheral and pin configuration must match the connection used by the Wi-Fi module.

### User LED

The supplied SBB selects `BSP_IO_PORT_02_PIN_07` as `USER_LED` for both FPB-RA4E2 and FPB-RA6E2. If another board or product uses a different LED or output, update the applicable board configuration or LED-control integration.

### Wi-Fi configuration

The SBB uses `g_wifi_onchip_silex_cfg` when opening the Wi-Fi module. The target project must provide the Wi-Fi OnChip Silex configuration required by the supplied code.

Wi-Fi credentials are provided interactively by the reference application. A product that obtains credentials by another mechanism should replace that demo-level input workflow while providing the required SSID, security type, and password to the SBB.

### Network behavior and timing

The supplied network constants include:

- `PING_COUNT`: 4
- `PING_INTERVAL_MS`: 100 ms
- `WIFI_MAX_TRY`: 5
- TCP send timeout: 400 ms
- TCP receive timeout: 15,000 ms
- `BUFF_LEN`: 32 bytes
- Maximum Wi-Fi scan results: 10

Review these values if the target system requires different retry, timeout, buffer, or scanning behavior.

### TCP server

The reference application obtains the server IP address and port from the terminal. Replace this UI behavior if the product stores or discovers the endpoint differently.

### Tasking

The reference source states that `wifi_thread_entry.c` calls `wifi_example_app_run()`. When the example exits after losing network connectivity, it calls `vTaskDelay(1)` continuously. The supplied files do not specify a task priority value.

### Console

The reference application depends on `console/uart_console.h` and standard input/output functions for its interactive workflow. This layer can be replaced if the product does not require a terminal-driven user interface.

## 7. Dependencies

### Renesas FSP and Wi-Fi middleware

The implementation depends on the Wi-Fi OnChip Silex middleware APIs, including:

- `rm_wifi_onchip_silex_open()`
- `rm_wifi_onchip_silex_close()`
- `rm_wifi_onchip_silex_scan()`
- `rm_wifi_onchip_silex_connect()`
- `rm_wifi_onchip_silex_socket_connected()`
- `rm_wifi_onchip_silex_ip_addr_get()`
- `rm_wifi_onchip_silex_dns_query()`
- `rm_wifi_onchip_silex_ping()`
- `rm_wifi_onchip_silex_socket_create()`
- `rm_wifi_onchip_silex_tcp_connect()`
- `rm_wifi_onchip_silex_socket_status_get()`
- `rm_wifi_onchip_silex_send()`
- `rm_wifi_onchip_silex_recv()`
- `rm_wifi_onchip_silex_disconnect()`

The code also uses the generated `g_wifi_onchip_silex_cfg` configuration.

### FSP/BSP resources

The LED implementation uses:

- `R_IOPORT_PinWrite()`
- `g_ioport.p_ctrl`
- `BSP_IO_PORT_02_PIN_07` for FPB-RA4E2 and FPB-RA6E2

The reference application also uses `R_FSP_VersionGet()`.

### Console and runtime dependencies

The reference application uses:

- `console/uart_console.h`
- `uart_console_init()`
- `uart_console_flush()`
- Standard input/output functions including `printf()` and `fgets()`
- `vTaskDelay()` after the demonstration exits

### Hardware and network resources

The demonstrated setup requires:

- FPB-RA4E2 or FPB-RA6E2
- Renesas Silex UART Pmod connected to PMOD1
- Wi-Fi access point
- Network connectivity used for the demonstrated DNS and ping operations
- PC/laptop on the same Wi-Fi access point for the TCP-server demonstration
- On-board user LED
- UART terminal connection

## 8. Demonstrated Behavior and Performance

The supplied project demonstrates:

- Scanning for Wi-Fi access points.
- Manual entry of SSID and security configuration.
- Support in the reference UI for Open, WPA, and WPA2 selections.
- Connection to a Wi-Fi access point and retrieval/display of the MCU IP address.
- DNS query or IP-address validation.
- Ping operation using 4 pings with a configured 100 ms interval.
- IPv4 TCP client socket creation.
- Connection to a user-entered TCP server IP address and port.
- Transmission of an initial `Hello` message after TCP connection.
- Receipt of `on` and `off` commands from the TCP server.
- On-board LED control in response to those commands.
- `Success` acknowledgment for valid commands and `Invalid` for other commands.
- Preservation of the current LED state for invalid commands.

The TCP send operation uses a 400 ms timeout and the receive operation uses a 15-second timeout. TCP connection attempts use `WIFI_MAX_TRY`, which is defined as 5.

The supplied files do not provide throughput, latency, memory-footprint, CPU-load, or RF-performance measurements.

## 9. File Modification Guidance

### SBB implementation files

- `wifi_ep.c`
- `wifi_ep.h`
- `tcp_socket.c`
- `tcp_socket.h`

These contain the Wi-Fi and network functionality used by the reference application. Product-specific changes to these files create a local SBB variant that must be reconciled when a newer SBB version is adopted.

### Demo/reference application files

- `wifi_example_app.c`
- `wifi_example_app.h`

These implement the terminal-oriented demonstration, including credential collection, access-point selection, ping-target input, TCP-server input, LED command interpretation, terminal output, and demonstration error handling. A product can replace this application layer while retaining the applicable SBB services.

### FSP-generated and configuration dependencies

The supplied source includes `hal_data.h` and uses `g_wifi_onchip_silex_cfg` and `g_ioport`. These are project/FSP configuration dependencies rather than functionality implemented by the SBB source files listed above.

## 10. Adopting a Newer SBB Release After Local Modifications

The supplied files do not define an automatic upgrade or source-merge mechanism for locally modified SBB code.

To reduce future integration work, keep product-specific workflows, credential sources, endpoint selection, command processing, and user-interface behavior outside the SBB implementation modules where practical. The existing separation between `wifi_ep.*`, `tcp_socket.*`, and `wifi_example_app.*` provides a natural boundary for doing this.

If the SBB implementation itself is modified, record those changes. When moving to a newer SBB release, compare the updated Wi-Fi/TCP modules and FSP configuration assumptions against the locally modified version, reapply only the product-specific changes still required, and revalidate Wi-Fi connection, DNS/ping, TCP connectivity, data transfer, and any product-specific hardware behavior.

## Reference Application Operation

1. Connect the Renesas Silex UART Pmod to PMOD1, aligning pin 1 correctly.
2. On FPB-RA4E2, close `E6` and open `E5` for PMOD1 UART operation.
3. Connect the board USB debug port to the host PC.
4. Ensure the PC/laptop is connected to the same Wi-Fi access point and that the access-point credentials are available.
5. Configure the serial terminal for `115200`, 8 data bits, no parity, 1 stop bit, no flow control, CR receive/transmit newline, and local echo.
6. Run the application and either scan for an access point or manually enter its credentials.
7. Enter a URL or IP address for the DNS/ping demonstration.
8. Start a TCP server on the PC. The supplied README recommends a free port greater than `10,000`.
9. Enter the TCP server IP address and port in the MCU terminal.
10. After the connection is established, send `on` or `off` from the server to control the board LED.
11. Observe the LED status and acknowledgment messages on the terminal/server.

For the ping demonstration, the supplied README instructs the user to enable the Windows Defender Firewall inbound rule `File and Printer Sharing (Echo Request - ICMPv4-In)` on the host PC when accepting ping requests from the RA MCU to the TCP server.

## Source File Map

| File | Role |
|---|---|
| `wifi_ep.c` | Wi-Fi module initialization, configuration, scanning, connection status, IP-address retrieval, string normalization, LED control, and deinitialization. |
| `wifi_ep.h` | Wi-Fi constants, board LED selection, Wi-Fi data structure, shared state, and public Wi-Fi function declarations. |
| `tcp_socket.c` | DNS, ping, TCP socket creation/connection, connection checking, send/receive operations, and socket deinitialization. |
| `tcp_socket.h` | Network constants, timeouts, socket data structure, byte-order conversion, and public network function declarations. |
| `wifi_example_app.c` | Demo/reference workflow for Wi-Fi setup, ping, TCP client connection, terminal interaction, and LED command processing. |
| `wifi_example_app.h` | Reference-application interface plus terminal banner, menu, guide, and error strings. |
