# 🧙‍♂️ SMM - Secret Manager Maker CLI

Welcome to **SMM** — the **Secret Manager Maker CLI** that transforms your scattered environment configs, YAMLs, and JSONs into polished AWS Secrets Manager commands. Whether you use **AWS CLI** or **Teleport (tsh)**, `smm` handles the heavy lifting, so you can focus on the fun stuff. 🎩✨

## 🚀 Features

- **Multi-format Support**: Converts environment variables, YAML, JSON files, or any other config format into AWS Secrets Manager commands.
- **Flexible Formatting**: Choose between **AWS CLI** or **Teleport (tsh)** formats with a single configuration setting.
- **Environment-Aware**: Process secrets for different environments (dev, staging, prod) and AWS regions effortlessly.
- **Automated Command Generation**: Generate the exact commands needed for pushing secrets to AWS, no more manual typing!
- **Cross-Platform**: Works seamlessly on **macOS** and **Linux**.
- **User-Friendly**: Easy-to-read error messages and warnings help you manage your secrets with confidence.

## 🛠 Installation

Get started with `smm` by cloning the repository and making the script executable:

```bash
# Clone the repository
git clone https://github.com/your-repo/smm-cli.git
cd smm-cli

# Make the script executable
chmod +x smm

# Optionally, move it to a location in your PATH for easier access
sudo mv smm /usr/local/bin/
```

## 📖 Usage

1. **Convert & Push your Configs to AWS:**

```bash
smm convert app_config.json --region us-west-2 --env prod
```

2. **Specify Line Ranges (if needed):**

```bash
smm convert .env --lines 5-10 --region us-east-1
```

3. **Batch Process Multiple Environments:**

```bash
smm convert secrets.yaml --envs dev,staging,prod --region us-east-1
```

### 💥 Example Output (AWS CLI Format):

```bash
aws secretsmanager create-secret --name dev/k8s/DATABASE_URL --description "Auto-created secret via smm" --secret-string "jdbc://dev-db"
```

### 💥 Example Output (Teleport Format):

```bash
tsh aws secretsmanager create-secret --name 'dev/k8s/DATABASE_URL' --secret-string "jdbc://dev-db" --region us-west-2
```

## 🌍 Cross-Platform Support

Good news, adventurer! Whether you're on **macOS** or **Linux**, `smm` is there for you.

- **macOS**: Works flawlessly, just install it via the instructions above or with `brew` (coming soon!).
- **Linux**: Supports all major distros like Debian, Ubuntu, CentOS, and Red Hat.

## 🛠 Contributing

We love contributors! 💖 Want to add a feature or fix a bug? Here's how you can help:

1. Fork the repo 🏴‍☠️
2. Create a new branch (`git checkout -b my-new-feature`) 🌿
3. Make your magical changes and commit (`git commit -am 'Added magic feature'`) 🧙‍♂️
4. Push to the branch (`git push origin my-new-feature`) 🚀
5. Open a Pull Request with a smile 😊

## 📝 License

This project is licensed under the **MIT License**. Feel free to use and modify it, but make sure you give us some credit (and maybe buy us a coffee ☕)!

---

🎩 **SMM your secrets into AWS!** Your configs deserve the best. 💼

---

And remember, "A good secret is one that is well managed.™"
