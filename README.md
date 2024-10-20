# 🔐 Secret Manager Maker (smm) CLI

`Secret Manager Maker (smm)`  CLI 🛠️ tool helps you automate 🤖 the process of converting 🔄 environment 🌍 configuration files 📂 into AWS Secrets Manager 🔑 commands. Whether you are working with development 🛠️, test 🧪, or production 🏭 environments, this tool allows you to push secrets 🤫 directly to AWS Secrets Manager with ease. You can also choose between AWS CLI and Teleport (tsh) 🚀 command formats.

## 🌟 Features

- 📖 Automatically reads your environment configuration from YAML, 📄 or custom config files.
- 🔄 Converts the configuration into AWS Secrets Manager or Teleport secrets creation commands.
- 🌍 Supports multiple environments like development 🛠️, testing 🧪, and production 🏭.
- 🎯 Allows users to specify line ranges, custom key names 🔑, and output scripts 📜.
- 🐛 Debug mode for troubleshooting.

## ⚙️ Installation

1. 🌀 **Clone the repository**:
   ```bash
   git clone https://github.com/yourusername/smm-cli.git
   cd smm-cli
   ```

2. 🔧 **Make the script executable**:
   ```bash
   chmod +x smm
   ```

3. ▶️ **Run the CLI**:
   ```bash
   ./smm --help
   ```

## 🚀 Usage

```bash
./smm [OPTIONS]
```

### ⚙️ Options

| ⚙️ Option           | 📋 Description                                                              |
| ------------------- | -------------------------------------------------------------------------- |
| `-h`, `--help`      | 📖 Show help message and exit.                                              |
| `-D`                | 🐞 Enable debug mode for verbose logging.                                   |
| `--lines RANGE`     | 🔢 Specify a range of line numbers to read from the config file (e.g., 5-10).|
| `--keyname NAME`    | 🔑 Use a custom keyName for the secrets instead of using the config.        |
| `--config FILE`     | 📂 Specify a custom configuration file. Default is `smm.yaml`.              |
| `--output FILE`     | 💾 Specify the output script file. Default is `create_secrets.sh`.          |

### ✨ Example Command
1. **Generate AWS CLI commands with default config**:
   By default, the tool reads from `smm.yaml` and generates the secrets creation commands in the default output file (`create_secrets.sh`).
   ```bash
   ./smm
   ```

2. **Enable debug mode to track execution**:
   ```bash
   ./smm -D
   ```

   The debug mode will display details about the configuration file being used, the output script being generated, and the values extracted from the YAML.

3. **Specify a custom configuration file**:
   Instead of the default `smm.yaml`, you can specify a custom configuration file.
   ```bash
   ./smm --config custom_config.yaml
   ```

4. **Generate Teleport (tsh) commands**:
   By changing the `format` in your `smm.yaml` to `tsh`, you can create Teleport CLI commands:
   ```yaml
   format: tsh  # in smm.yaml
   ```
   Then run:
   ```bash
   ./smm
   ```

5. **Custom output file**:
   Specify a custom output file for the generated script:
   ```bash
   ./smm --output my_secrets_script.sh
   ```

### Combining Multiple Options

1. **Custom keyName with line range**:
   If you want to specify a custom key name while processing only a specific range of lines from the config file:
   ```bash
   ./smm --keyname MY_SECRET_KEY --lines 5-10
   ```

   In this example, `MY_SECRET_KEY` will be used for each secret, and only lines 5 to 10 of the config file will be read.

2. **Debug mode with custom keyName and custom output**:
   Combine debug mode to get detailed output, specify a custom key name, and save the generated commands to a custom file:
   ```bash
   ./smm -D --keyname MY_CUSTOM_KEY --output secrets_script.sh
   ```

   This is useful when you need to debug the script and ensure the correct key name and output file are being used.

3. **Custom config file, custom output, and range of lines**:
   Use a specific configuration file, output to a custom script, and process only specific lines:
   ```bash
   ./smm --config custom_config.yaml --output custom_secrets.sh --lines 10-20
   ```

   This reads the configuration from `custom_config.yaml`, writes the generated commands to `custom_secrets.sh`, and only processes lines 10 to 20 in the configuration file.

4. **Mix of AWS CLI format and a specific range of lines**:
   You can specify that you only want the secrets to be created for a subset of your configuration file, using AWS CLI format:
   ```yaml
   format: aws  # in smm.yaml
   ```
   Then run:
   ```bash
   ./smm --lines 10-30
   ```

   This will generate AWS CLI commands for secrets defined between lines 10 and 30 of the configuration.

5. **Create secrets for production environment only**:
   Suppose you only want to process lines containing production-related secrets and use a custom key:
   ```bash
   ./smm --keyname PROD_SECRET_KEY --lines 50-70 --output prod_secrets.sh
   ```

   This reads the configuration file, extracts secrets from lines 50 to 70, uses the `PROD_SECRET_KEY` key name, and generates the output in `prod_secrets.sh`.

### Advanced Scenarios

1. **Multiple secrets creation with custom script for each environment**:
   You can create separate scripts for each environment (dev, test, prod) by modifying the environments in your `smm.yaml` and specifying different output files:
   ```yaml
   environments:
     - name: project_name-dev  # Development environment
       region: us-west-2
     - name: project_name-test  # Test environment
       region: us-west-2
     - name: project_name-prod  # Production environment
       region: us-east-1
   ```

   Run for the development environment:
   ```bash
   ./smm --keyname DEV_SECRET_KEY --output dev_secrets.sh
   ```

   Run for the production environment:
   ```bash
   ./smm --keyname PROD_SECRET_KEY --output prod_secrets.sh
   ```

2. **Using different configurations for staging and production**:
   You can maintain separate configuration files for different environments (e.g., `staging.yaml` and `production.yaml`) and specify them as needed:
   ```bash
   ./smm --config staging.yaml --output staging_secrets.sh
   ./smm --config production.yaml --output production_secrets.sh
   ```

### Custom Configuration Example

Here's an example of what your custom configuration YAML might look like:

```yaml
# custom_config.yaml
configFile: "prod/hub/config/appglobal.conf"  # Path to the production config file
outputScript: "prod_secrets.sh"  # The output file where the AWS CLI/Teleport commands will be written

environments:
  - name: project_name-prod  # Production environment
    region: us-east-1

format: aws  # Options: "tsh" (Teleport) or "aws" (AWS CLI)
```

Then you can run:
```bash
./smm --config custom_config.yaml --output custom_script.sh
```

### Debugging

To enable detailed debug logging to trace the behavior of the script and see the values being processed, use the `-D` option:
```bash
./smm -D --config smm.yaml --output my_debug_script.sh
```

This provides detailed information about each step, such as:
- Configuration values being extracted.
- Output scripts being generated.
- Secrets and key names being processed.

---

These additional examples demonstrate how you can combine different options in the `smm` CLI to fit various real-world use cases.

## 📝 Configuration (`smm.yaml`)

`smm` uses a YAML configuration file 📄 to define environment settings ⚙️. The default file is `smm.yaml` and looks like this:

```yaml
# smm.yaml - Configuration for the Secret Manager Maker CLI

configFile: "secrets.conf"  # The path to your application config file
outputScript: "create_secrets.sh"  # The output file where the AWS CLI/Teleport commands will be written

environments:
  - name: project_name-dev  # Development environment
    region: us-west-2
  - name: project_name-test  # Test environment
    region: us-west-2
  - name: project_name-prod  # Production environment
    region: us-east-1

format: tsh  # Options: "tsh" (Teleport) or "aws" (AWS CLI)
```

### 🔑 Key Components:

- **configFile**: The path 🛤️ to the config file where your key-value pairs for secrets 🤫 are defined.
- **outputScript**: The file where the generated AWS CLI/Teleport commands 🚀 will be saved 💾.
- **environments**: Define your environment names 🛠️ (e.g., dev, test, prod) and their associated regions 🌍.
- **format**: Choose between `tsh` for Teleport commands or `aws` for AWS Secrets Manager commands.

## 🌍 Cross-Platform Support

`smm` is designed for Unix-based systems like macOS 🍏 and Linux 🐧. Simply run the script in your terminal 💻, and it will function as expected. Ensure you have access to AWS CLI and/or Teleport CLI based on your configuration needs.

## 📋 Example Output

Here’s an example of what the generated script might look like when using AWS CLI:
```bash
#!/bin/bash

aws secretsmanager create-secret --name project_name-dev/k8s/MY_CUSTOM_KEY --secret-string "secret_value" --region us-west-2
aws secretsmanager create-secret --name project_name-test/k8s/MY_CUSTOM_KEY --secret-string "another_secret_value" --region us-west-2
aws secretsmanager create-secret --name project_name-prod/k8s/MY_CUSTOM_KEY --secret-string "prod_secret_value" --region us-east-1
```

## 🐞 Debugging

Enable the debug mode 🐛 using the `-D` option to get detailed output of the script execution:
```bash
./smm -D --config myconfig.yaml
```

## 🤝 Contributing

1. 🍴 **Fork the repository**.
2. 🌿 **Create your feature branch**: `git checkout -b feature/your-feature-name`.
3. ✏️ **Commit your changes**: `git commit -m 'Add some feature'`.
4. 🚀 **Push to the branch**: `git push origin feature/your-feature-name`.
5. 📬 **Open a pull request**.

## 📜 License

This project is licensed under the MIT License 📄.

## 🚀 Get Started

Get started with `smm` by running:
```bash
./smm --help
```

---

This `README.md` provides all the necessary details on how to use the `smm` CLI tool, its options, and how to configure and run it effectively.



🎩 **SMM your secrets into AWS!** Your configs deserve the best. 💼

---

And remember, "A good secret is one that is well managed.™"

