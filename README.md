# IMC Accident MCP Server

**A Model Context Protocol (MCP) server for managing accident and customer data within the IMC ecosystem, built with Spring AI and Spring Boot.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Java 21+](https://img.shields.io/badge/java-21+-blue.svg)](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)
[![Maven 3.6+](https://img.shields.io/badge/maven-3.6+-red.svg)](https://maven.apache.org/download.cgi)

This server provides a foundation for building MCP-enabled applications that interact with accident and customer-related functionalities. It supports both STDIO transport (for Claude Desktop integration) and SSE transport (for web-based clients).

For more information on the underlying framework, see the [MCP Server Boot Starter](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-starter-docs.html) reference documentation.

## 📜 Table of Contents
- [Overview](#-overview)
- [Available Tools](#-available-tools)
- [🚀 Quick Start](#-quick-start)
- [🤖 Claude Desktop Integration](#-claude-desktop-integration)
- [⚙️ Configuration](#️-configuration)
- [🏗️ Architecture](#️-architecture)
- [🛠️ Development](#️-development)
- [🧪 Client Examples](#-client-examples)
- [📦 Dependencies](#-dependencies)
- [🤝 Contributing](#-contributing)
- [📚 Additional Resources](#-additional-resources)
- [License](#-license)

## ✨ Overview

This IMC Accident MCP server demonstrates:
- Integration with `spring-ai-mcp-server-webflux-spring-boot-starter` for MCP capabilities.
- Dual transport support: **STDIO** (for CLI/desktop clients like Claude Desktop) and **SSE** (Server-Sent Events for web-based clients).
- Automatic tool registration using Spring AI's `@Tool` annotation, enabling AI models to interact with accident and customer data.
- Clean separation of concerns with a dedicated `ToolsService` for business logic.
- Production-ready logging configuration.
- Comprehensive test coverage.

## 🛠️ Available Tools

This server exposes tools that can be invoked by an AI model to query customer and accident information:

### 👤 queryCustomer
- **Description**: Query customer information by customer ID. Returns customer contact details and address.
- **Parameters**:
  - `customerId` (Integer): The customer ID to search for.
- **Example**: `queryCustomer(123)`

### 💥 queryAccidents
- **Description**: Query accident information by customer ID. Returns all accidents where the customer was the driver.
- **Parameters**:
  - `customerId` (Integer): The customer ID (driver ID) to search accidents for.
- **Example**: `queryAccidents(123)`

## 🚀 Quick Start

### Prerequisites
- Java 21+
- Maven 3.6+

### Building the Project
```bash
./mvnw clean install
```

### Running the Server

#### For Claude Desktop (STDIO Mode)
```bash
java -Dspring.profiles.active=stdio -jar target/mcp-server-0.0.1-SNAPSHOT.jar
```

#### For Web Clients (SSE Mode - Default)
```bash
java -jar target/mcp-server-0.0.1-SNAPSHOT.jar
```
Server will be available at `http://localhost:8080/mcp/message`

### Testing the Server
Use the included test script:
```bash
# Test STDIO mode
./test-mcp.sh --stdio --test-tools

# Test SSE mode
./test-mcp.sh --sse --test-tools

# Build and test both modes
./test-mcp.sh --build --both --test-tools
```

## 🤖 Claude Desktop Integration

### 1. Add to Claude Desktop Configuration

Add this to your Claude Desktop `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "mcp-server": {
      "command": "java",
      "args": [
        "-Dspring.profiles.active=stdio",
        "-jar",
        "/absolute/path/to/mcp-server/target/mcp-server-0.0.1-SNAPSHOT.jar"
      ]
    }
  }
}
```

### 2. Restart Claude Desktop

The tools `capitalizeText` and `calculate` will be available for use.

### 3. Test the Tools

Try asking Claude Desktop:
- "Query customer with ID 1"
- "Find accidents for customer ID 2"

## ⚙️ Configuration

The server uses profile-based configuration:

### STDIO Profile (`application-stdio.properties`)
```properties
# STDIO Transport for Claude Desktop
spring.ai.mcp.server.stdio=true
spring.main.web-application-type=none
spring.main.banner-mode=off

# Logging to file only (console interferes with MCP protocol)
logging.level.root=OFF
logging.config=classpath:logback-stdio.xml
```

### SSE Profile (`application-sse.properties`)
```properties
# SSE Transport for Web Clients
spring.ai.mcp.server.stdio=false
server.port=8080

# MCP endpoint
spring.ai.mcp.server.sse-message-endpoint=/mcp/message
```

## 🏗️ Architecture

### Core Components

- **`McpServerApplication`**: Main Spring Boot application with tool registration.
- **`ToolsService`**: Service containing MCP tools with `@Tool` annotations, where accident and customer-related business logic would reside.
- **Transport Layer**: Automatic Spring AI MCP transport configuration.
- **Configuration**: Profile-based setup for different deployment modes.

### Project Structure
```
src/
├── main/java/com/baskettecase/mcpserver/
│   ├── McpServerApplication.java      # Main application
│   ├── ToolsService.java              # MCP tools implementation (e.g., for accident/customer data)
│   └── model/
│       ├── Accident.java              # Data model for Accident entity
│       └── Customer.java              # Data model for Customer entity
├── main/resources/
│   ├── application.properties         # Base configuration
│   ├── application-stdio.properties   # STDIO transport config
│   ├── application-sse.properties   # SSE transport config
│   └── logback-stdio.xml              # STDIO logging config
└── test/java/
    ├── ToolsServiceTest.java          # Comprehensive tool tests
    └── org/springframework/ai/mcp/sample/client/
        ├── SampleClient.java          # Example MCP client
        ├── ClientStdio.java           # STDIO client example
        └── ClientSse.java             # SSE client example
```

## 🛠️ Development

### Adding New Tools

Add methods to `ToolsService` with the `@Tool` annotation to expose new functionalities to the AI model:

```java
@Tool(description = "Your tool description")
public String yourTool(String parameter) {
    // Your implementation for accident/customer data interaction
    return "result";
}
```

### Running Tests
```bash
./mvnw test
```

### Viewing Logs

- **STDIO mode**: Logs go to `/tmp/mcp-server-stdio.log`
- **SSE mode**: Logs go to console and `./target/mcp-server-sse.log`

## 🧪 Client Examples

### Manual MCP Client (STDIO)
```java
var stdioParams = ServerParameters.builder("java")
    .args("-Dspring.profiles.active=stdio", "-jar", "target/mcp-server-0.0.1-SNAPSHOT.jar")
    .build();

var transport = new StdioClientTransport(stdioParams);
var client = McpClient.sync(transport).build();
```

### Manual MCP Client (SSE)
```java
var transport = new WebFluxSseClientTransport(
    WebClient.builder().baseUrl("http://localhost:8080"));
var client = McpClient.sync(transport).build();
```

## 📦 Dependencies

Key dependencies include:

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-mcp-server-webflux-spring-boot-starter</artifactId>
    <version>1.1.0-SNAPSHOT</version>
</dependency>
```

This starter provides:
- Reactive and STDIO transport support
- Auto-configured MCP endpoints
- Tool callback registration
- Spring WebFlux integration

## 🤝 Contributing

This project serves as a foundation for the IMC Accident MCP server. Feel free to:
- Add new tools to `ToolsService` for accident and customer management.
- Extend transport configurations.
- Improve test coverage.
- Add new MCP capabilities.

## 📚 Additional Resources

- [Spring AI Documentation](https://docs.spring.io/spring-ai/reference/)
- [MCP Server Boot Starter Docs](https://docs.spring.io/spring-ai/reference/api/mcp/mcp-server-boot-starter-docs.html)
- [Model Context Protocol Specification](https://modelcontextprotocol.github.io/specification/)
- [Claude Desktop MCP Guide](https://claude.ai/mcp)

## License

This project is provided as-is for educational and development purposes.