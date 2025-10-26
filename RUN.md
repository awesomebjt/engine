# Running Open Integration Engine

This guide describes how to run the Open Integration Engine application after building it. For build instructions, see [BUILD.md](BUILD.md).

## Table of Contents
- [Prerequisites](#prerequisites)
- [Starting the Server](#starting-the-server)
- [Launching the Frontend Application](#launching-the-frontend-application)
- [Configuration](#configuration)
- [Default Credentials](#default-credentials)
- [Troubleshooting](#troubleshooting)

## Prerequisites

1. **Complete the build** - Follow the instructions in [BUILD.md](BUILD.md) to build the project
2. **Java 8 with JavaFX** - Same JDK used for building (Zulu 8.0.462.fx recommended)
3. **Built artifacts** - The `server/setup/` directory should be populated with JARs and configuration files

## Starting the Server

Before launching the frontend application, you must start the Mirth Connect Server:

### Option 1: Using the Server Launcher

```bash
cd server/setup
java -jar mirth-server-launcher.jar
```

### Option 2: Development Mode

For development and testing:

```bash
cd server
ant -f build.xml dev-run
```

### Option 3: Using the Launcher Script

```bash
cd server/setup
java -jar mirth-manager-launcher.jar
```

This launches the application manager GUI with a system tray icon for managing the server.

### Verify Server is Running

The server should be accessible at:
- **HTTP:** `http://localhost:8080`
- **HTTPS:** `https://localhost:8443`

Check the logs in `server/setup/logs/` if the server fails to start.

## Launching the Frontend Application

Once the server is running, you can launch the Mirth Connect Administrator (desktop client) using one of the following methods:

### Method 1: Web-Based Launch (Recommended for End Users)

This is the easiest method for launching the client application:

1. Open a web browser
2. Navigate to: `http://localhost:8080`
3. Click the **"Launch Mirth Connect Administrator"** button
4. The application will launch via Java Web Start (JNLP)
5. Login with default credentials (see below)

**Benefits:**
- No manual classpath configuration needed
- Application updates are automatically downloaded from the server
- Libraries are dynamically loaded from the server

**Note:** You may see a security warning when launching. This is normal for self-signed certificates.

### Method 2: Application Manager Launcher

Launch the GUI application manager which provides a system tray icon:

```bash
cd server/setup
java -jar mirth-manager-launcher.jar
```

**Optional:** Specify the Mirth installation directory:
```bash
java -jar mirth-manager-launcher.jar /path/to/mirth
```

The manager provides:
- System tray icon for easy access
- Server start/stop controls
- Quick launch of the administrator client

### Method 3: Direct Client JAR Execution

Run the client application directly with command-line parameters:

```bash
cd server/setup
java -cp "client-lib/*" com.mirth.connect.client.ui.Mirth https://localhost:8443 0.0.0 admin admin
```

**Command-line argument format:**
```
<server> <version> [<username> [<password> [-ssl [<protocols> [<ciphersuites>]]]]]
```

**Examples:**

Basic connection:
```bash
java -cp "client-lib/*" com.mirth.connect.client.ui.Mirth https://localhost:8443 0.0.0
```

With credentials:
```bash
java -cp "client-lib/*" com.mirth.connect.client.ui.Mirth https://localhost:8443 0.0.0 admin admin
```

With SSL protocol specification:
```bash
java -cp "client-lib/*" com.mirth.connect.client.ui.Mirth https://localhost:8443 0.0.0 -ssl TLSv1.3
```

With SSL and credentials:
```bash
java -cp "client-lib/*" com.mirth.connect.client.ui.Mirth https://localhost:8443 0.0.0 admin admin -ssl TLSv1.3
```

### Method 4: Development Launch Configuration

For IDE development (Eclipse):

1. Open the launch configuration: `client/Mirth Connect Client.launch`
2. Main class: `com.mirth.connect.client.ui.Mirth`
3. Program arguments: `https://localhost:8443 0.0.0 admin admin`
4. Run the configuration

### Method 5: Command-Line Interface (CLI)

For non-GUI operations and scripting:

```bash
cd server/setup
java -jar mirth-cli-launcher.jar
```

This launches an interactive shell for managing Mirth Connect via command-line.

## Configuration

### Server Configuration

**Configuration file:** `server/setup/conf/mirth.properties`

Key settings:
```properties
# HTTP port for web-based client launch
http.port = 8080

# HTTPS port for client-server communication
https.port = 8443

# SSL/TLS protocols
https.client.protocols = TLSv1.3,TLSv1.2

# Keystore location
keystore.path = ${dir.appdata}/keystore.jks
```

### Client Configuration

**Configuration file:** `server/setup/conf/mirth-cli-config.properties`

Default settings:
```properties
address = https://127.0.0.1:8443
user = admin
password = admin
version = 0.0.0
```

### JVM Options

**Configuration files:**
- `server/setup/conf/base_includes.vmoptions` - Base JVM settings
- `server/setup/conf/default_modules.vmoptions` - Java 9+ module system settings
- `server/setup/conf/custom.vmoptions` - User customizations

For Java 9+ environments, the client requires specific module system arguments:
```
--add-modules=java.sql.rowset
--add-exports=java.base/sun.security.provider=ALL-UNNAMED
--add-opens=java.base/java.util=ALL-UNNAMED
```

These are configured automatically when using the JNLP launcher.

## Default Credentials

**Username:** `admin`
**Password:** `admin`

**Security Note:** Change the default password after first login in a production environment.

## Troubleshooting

### Server Won't Start

**Check if port is already in use:**
```bash
# Linux/Mac
lsof -i :8080
lsof -i :8443

# Windows
netstat -ano | findstr :8080
netstat -ano | findstr :8443
```

**Check server logs:**
```bash
tail -f server/setup/logs/mirth.log
```

**Verify Java version:**
```bash
java -version
# Should show Java 8 with JavaFX (e.g., 8.0.462.fx-zulu)
```

### Client Can't Connect to Server

**Error message:** "There was an error connecting to the server at the specified address."

**Solutions:**

1. **Verify server is running:**
   ```bash
   curl -k https://localhost:8443
   ```

2. **Check server address:**
   - Default: `https://localhost:8443`
   - Verify the address in `conf/mirth-cli-config.properties`

3. **Check firewall settings:**
   - Ensure ports 8080 and 8443 are not blocked
   - Add firewall exceptions if needed

4. **Verify SSL certificate:**
   - Self-signed certificates may cause connection issues
   - Check keystore configuration in `conf/mirth.properties`

### Java Web Start (JNLP) Issues

**Security warning appears:**
- This is normal for self-signed certificates
- Click "Run" or "Accept" to proceed

**Application doesn't launch:**
- Verify Java Web Start is enabled in your Java installation
- Check browser console for JNLP download errors
- Try direct JAR execution method instead

### Memory Issues

**Increase JVM heap size:**

For the server:
```bash
export JAVA_OPTS="-Xms512m -Xmx2048m"
java -jar mirth-server-launcher.jar
```

For the client (when using direct execution):
```bash
java -Xms256m -Xmx512m -cp "client-lib/*" com.mirth.connect.client.ui.Mirth https://localhost:8443 0.0.0
```

### Port Already in Use

**Change default ports:**

Edit `server/setup/conf/mirth.properties`:
```properties
http.port = 9090
https.port = 9443
```

Then update client connection address accordingly.

### Database Connection Issues

**Create Derby database (if needed):**
```bash
cd server
ant -f build.xml create-derby-db
```

Check database configuration in `server/setup/conf/mirth.properties`.

## Additional Resources

- **Build Instructions:** [BUILD.md](BUILD.md)
- **Project README:** [README.md](README.md)
- **Server Logs:** `server/setup/logs/`
- **Configuration Files:** `server/setup/conf/`
- **Documentation:** `server/docs/README.txt`

## Quick Reference

### Common Commands

```bash
# Start server
cd server/setup && java -jar mirth-server-launcher.jar

# Launch client via web browser
open http://localhost:8080

# Launch manager GUI
cd server/setup && java -jar mirth-manager-launcher.jar

# Launch CLI
cd server/setup && java -jar mirth-cli-launcher.jar

# Direct client execution
cd server/setup && java -cp "client-lib/*" com.mirth.connect.client.ui.Mirth https://localhost:8443 0.0.0 admin admin
```

### Default Endpoints

| Service | URL | Purpose |
|---------|-----|---------|
| Web Launcher | `http://localhost:8080` | Launch client via browser |
| API Endpoint | `https://localhost:8443` | Client-server communication |
| Server Logs | `server/setup/logs/` | Log files location |

### File Locations

| Component | Location |
|-----------|----------|
| Server JARs | `server/setup/server-lib/` |
| Client JARs | `server/setup/client-lib/` |
| Extensions | `server/setup/extensions/` |
| Configuration | `server/setup/conf/` |
| Logs | `server/setup/logs/` |
| Database | `server/setup/appdata/` |
