#!/bin/bash

# smm - Secret Manager Maker CLI
# This script processes environment configs, YAML files, and converts them to AWS Secrets Manager CLI commands.

# Initialize variables
DEBUG=false
LINE_RANGE=""
CONFIG_FILE=""
OUTPUT_SCRIPT=""
FORMAT=""
inputKeyName=""

# Function to display help message
show_help() {
  cat <<EOF
Usage: smm [OPTIONS]

Secret Manager Maker CLI - Automate pushing environment configs to AWS Secrets Manager.

Options:
  -h, --help        Show this help message and exit
  -D                Enable debug mode
  --lines RANGE     Specify line numbers or range to read from config file (e.g., 5-10)
  --keyname NAME    Use a specific keyName for secrets (instead of reading from config)
  --config FILE     Specify the configuration file (default: smm.yaml)
  --output FILE     Specify the output script file (default: create_secrets.sh)
EOF
}

# Function to print debug information
debug() {
  if [ "$DEBUG" = true ]; then
    echo "DEBUG: $1"
  fi
}

# Function to create smm.yaml if it doesn't exist
create_smm_yaml() {
  if [[ ! -f "smm.yaml" ]]; then
    echo "smm.yaml not found, creating a basic configuration file..."
    cat <<EOL > smm.yaml
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
EOL
    echo "smm.yaml created with default values. Please modify it as necessary."
  fi
}

# Function to validate the smm.yaml file
validate_yaml_file() {
  if [[ ! -f "$CONFIG_FILE" ]]; then
    echo "Error: $CONFIG_FILE file not found!"
    exit 1
  fi

  # Ensure smm.yaml contains necessary fields
  if ! grep -q 'configFile' "$CONFIG_FILE" || \
     ! grep -q 'outputScript' "$CONFIG_FILE" || \
     ! grep -q 'environments' "$CONFIG_FILE" || \
     ! grep -q 'format' "$CONFIG_FILE"; then
    echo "Error: $CONFIG_FILE file is missing required fields!"
    exit 1
  fi
}

# Function to extract values from YAML manually, ignoring commented lines
extract_value_from_yaml() {
  local key=$1
  local value=$(awk -F': ' -v key="$key" '$1 ~ key {print $2}' "$CONFIG_FILE" | awk -F'#' '{print $1}' | xargs)

  if [[ -z "$value" ]]; then
    echo "Error: Value for $key not found or incorrectly formatted in $CONFIG_FILE!"
    exit 1
  fi

  echo "$value"
}

# Parse command-line options using get# Parse command-line options using getopts
# Parse command-line options using getopts
while getopts "hDl:k:c:o:" opt; do
  case "$opt" in
    h)
      show_help
      exit 0
      ;;
    D)
      DEBUG=true
      ;;
    l)
      LINE_RANGE="$OPTARG"
      ;;
    k)
      inputKeyName="$OPTARG"
      ;;
    c)
      CONFIG_FILE="$OPTARG"
      ;;
    o)
      OUTPUT_SCRIPT="$OPTARG"
      ;;
    *)
      show_help
      exit 1
      ;;
  esac
done

# Set default values if not provided
CONFIG_FILE=${CONFIG_FILE:-"smm.yaml"}
OUTPUT_SCRIPT=${OUTPUT_SCRIPT:-"create_secrets.sh"}

# Debugging output
debug "CONFIG_FILE: $CONFIG_FILE"
debug "OUTPUT_SCRIPT: $OUTPUT_SCRIPT"
debug "LINE_RANGE: $LINE_RANGE"
debug "inputKeyName: $inputKeyName"

# Create smm.yaml if it doesn't exist
create_smm_yaml

# Validate the smm.yaml file
validate_yaml_file

# Extract the required configuration values from YAML
CONFIG_FILE_PATH=$(extract_value_from_yaml "configFile")
OUTPUT_SCRIPT_PATH=$(extract_value_from_yaml "outputScript")
FORMAT=$(extract_value_from_yaml "format")

# Debugging output
debug "CONFIG_FILE_PATH: $CONFIG_FILE_PATH"
debug "OUTPUT_SCRIPT_PATH: $OUTPUT_SCRIPT_PATH"
debug "FORMAT: $FORMAT"

# Initialize the output script file
echo "#!/bin/bash" > "$OUTPUT_SCRIPT"
echo "" >> "$OUTPUT_SCRIPT"

# Function to handle line numbers or range
get_line_range() {
  if [[ -z "$LINE_RANGE" ]]; then
    read -p "Enter a specific line number or a range of line numbers (e.g., 5-10): " range
  else
    range=$LINE_RANGE
  fi

  if [[ $range =~ ^([0-9]+)-([0-9]+)$ ]]; then
    start_line=${BASH_REMATCH[1]}
    end_line=${BASH_REMATCH[2]}
  elif [[ $range =~ ^([0-9]+)$ ]]; then
    start_line=${BASH_REMATCH[1]}
    end_line=$start_line
  else
    echo "Invalid input. Please provide a single number or a range (e.g., 5-10)."
    exit 1
  fi
}

# Prompt the user if no argument is passed for line range
get_line_range

# Debugging output
debug "start_line: $start_line"
debug "end_line: $end_line"

# Extract environments manually from the YAML, ignoring commented lines
extract_environments() {
  awk '/environments:/ {flag=1; next} /format:/ {flag=0} flag {print}' "$CONFIG_FILE" | grep -v '^\s*#' | grep -v '^\s*$'
}

# Loop through each environment and region
extract_environments | while IFS= read -r line; do
  if [[ $line =~ name ]]; then
    envName=$(echo $line | awk -F': ' '{print $2}' | awk -F'#' '{print $1}' | xargs)
    debug "envName: $envName"
  elif [[ $line =~ region ]]; then
    region=$(echo $line | awk -F': ' '{print $2}' | awk -F'#' '{print $1}' | xargs)
    debug "region: $region"

    # Read the specific range of lines from the config file, skipping commented lines
    sed -n "${start_line},${end_line}p" "$CONFIG_FILE_PATH" | grep -v '^\s*//' | while IFS= read -r line
    do
      # Check if the line starts with a quoted value
      if [[ $line =~ ^([a-zA-Z0-9_]+)\ *=\ *\" ]]; then
        configKeyName="${BASH_REMATCH[1]}"
        keyValue="${line#*= }"  # Start accumulating the value
        while [[ ! $keyValue =~ \"[[:space:]]*$ ]]; do
          read -r next_line
          keyValue="$keyValue$next_line"
        done
        keyValue="${keyValue%\"*}\""  # Remove the trailing quote
      elif [[ $line =~ ^([a-zA-Z0-9_]+)\ *=\ *(.+)$ ]]; then
        configKeyName="${BASH_REMATCH[1]}"
        keyValue="${BASH_REMATCH[2]}"
      else
        echo "Warning: Skipping invalid line: '$line'"
        continue
      fi

      # Use the inputKeyName if provided, otherwise use the one from the config file
      keyName="${inputKeyName:-$configKeyName}"

      # Debugging output
      debug "configKeyName: $configKeyName"
      debug "keyValue: $keyValue"
      debug "keyName: $keyName"

      AWS_CLI_CMD="aws secretsmanager create-secret --name $envName/k8s/$keyName --secret-string \"$keyValue\" --region $region"
      # Generate commands based on the format option
      if [[ $FORMAT == "aws" ]]; then
        # Generate AWS CLI command
        echo $AWS_CLI_CMD >> "$OUTPUT_SCRIPT"
      elif [[ $FORMAT == "tsh" ]]; then
        # Generate Teleport (tsh) command
        TSH_CMD="tsh $AWS_CLI_CMD"
        echo $TSH_CMD >> "$OUTPUT_SCRIPT"
      else
        echo "Error: Unsupported format specified in $CONFIG_FILE. Use 'tsh' or 'aws'."
        exit 1
      fi
    done
    echo "" >> "$OUTPUT_SCRIPT"
  fi
done

# Make the output script executable
chmod +x "$OUTPUT_SCRIPT"

echo "Script $OUTPUT_SCRIPT has been created and is executable."
