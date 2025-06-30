# Intune Uploader - OIDC Enhanced Fork

This is a fork of the original [intune-uploader](https://github.com/almenscorner/intune-uploader) with enhanced OIDC (OpenID Connect) support for secure CI/CD pipelines.

## 🚀 Key Enhancements

### OIDC Authentication Support
- **Secure CI/CD**: No more client secrets in your pipelines
- **Federated Credentials**: Use Azure AD federated credentials with GitHub Actions
- **Automatic Token Management**: Seamless token handling for Microsoft Graph API

### Enhanced Processors
All processors now support both traditional client secret authentication and modern OIDC:

- **IntuneAppUploader** - Upload apps to Intune with OIDC support
- **IntuneAppCleaner** - Clean up old app versions
- **IntuneAppPromoter** - Promote apps through deployment rings
- **IntuneAppIconGetter** - Extract app icons automatically
- **IntuneScriptUploader** - Upload scripts to Intune
- **IntuneVTAppDeleter** - VirusTotal integration
- **IntuneTeamsNotifier** / **IntuneSlackNotifier** - Notifications

## 🔧 Usage

### Traditional Client Secret Method
```xml
<key>CLIENT_ID</key>
<string>your-client-id</string>
<key>CLIENT_SECRET</key>
<string>your-client-secret</string>
<key>TENANT_ID</key>
<string>your-tenant-id</string>
```

### Modern OIDC Method
```xml
<key>GRAPH_TOKEN</key>
<string>%GRAPH_TOKEN%</string>
```

The processor will automatically detect which method to use and handle authentication accordingly.

## 🛠 Installation

### For AutoPkg
```bash
# Clone this fork
git clone https://github.com/jon-mathias-tandberg/intune-uploader.git

# Copy processors to AutoPkg
sudo mkdir -p /Library/AutoPkg/autopkglib/IntuneUploader
sudo cp -r intune-uploader/IntuneUploader/* /Library/AutoPkg/autopkglib/IntuneUploader/
sudo chmod -R 755 /Library/AutoPkg/autopkglib/IntuneUploader/
```

### Dependencies
```bash
pip install -r IntuneUploader/requirements.txt
```

## 🔐 OIDC Setup

### 1. Azure AD Configuration
1. Create an App Registration in Azure AD
2. Configure federated credentials for GitHub Actions
3. Grant appropriate Microsoft Graph permissions

### 2. GitHub Actions
```yaml
- name: Azure Login
  uses: azure/login@v1
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}

- name: Get Graph Token
  run: |
    token=$(az account get-access-token --resource-type ms-graph --query accessToken -o tsv)
    echo "GRAPH_TOKEN=$token" >> $GITHUB_ENV
```

### 3. AutoPkg Recipe
```xml
<key>GRAPH_TOKEN</key>
<string>%GRAPH_TOKEN%</string>
```

## 📋 Example Recipe

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Identifier</key>
    <string>local.intune.Firefox</string>
    <key>Input</key>
    <dict>
        <key>GRAPH_TOKEN</key>
        <string>%GRAPH_TOKEN%</string>
        <key>display_name</key>
        <string>Firefox</string>
        <key>description</key>
        <string>Mozilla Firefox Browser</string>
        <key>publisher</key>
        <string>Mozilla</string>
        <key>bundleId</key>
        <string>org.mozilla.firefox</string>
        <key>bundleVersion</key>
        <string>%version%</string>
    </dict>
    <key>Process</key>
    <array>
        <dict>
            <key>Processor</key>
            <string>IntuneUploader/IntuneAppUploader</string>
        </dict>
    </array>
</dict>
</plist>
```

## 🔄 Migration from Original

This fork is fully compatible with the original intune-uploader. To migrate:

1. **Update processor paths** in your recipes:
   ```xml
   <!-- Old -->
   <string>com.github.almenscorner.intune-upload.processors/IntuneAppUploader</string>
   
   <!-- New -->
   <string>IntuneUploader/IntuneAppUploader</string>
   ```

2. **Add OIDC support** (optional):
   ```xml
   <key>GRAPH_TOKEN</key>
   <string>%GRAPH_TOKEN%</string>
   ```

3. **Keep existing variables** for backward compatibility:
   ```xml
   <key>CLIENT_ID</key>
   <string>your-client-id</string>
   <key>CLIENT_SECRET</key>
   <string>your-client-secret</string>
   <key>TENANT_ID</key>
   <string>your-tenant-id</string>
   ```

## 🐛 Troubleshooting

### Common Issues

1. **Token not found**
   - Ensure `GRAPH_TOKEN` is set in environment
   - Check Azure AD permissions

2. **Processor not found**
   - Verify installation path: `/Library/AutoPkg/autopkglib/IntuneUploader/`
   - Check file permissions

3. **Authentication failed**
   - Verify federated credentials in Azure AD
   - Check GitHub Actions permissions

### Debug Mode
```bash
autopkg run your-recipe.recipe -v
```

## 🤝 Contributing

This fork maintains compatibility with the original while adding OIDC support. Contributions are welcome!

## 📄 License

Same license as the original project. See [LICENSE](LICENSE) for details.

## 🔗 Links

- [Original intune-uploader](https://github.com/almenscorner/intune-uploader)
- [AutoPkg Documentation](https://github.com/autopkg/autopkg)
- [Microsoft Graph API](https://docs.microsoft.com/en-us/graph/)
- [Azure AD OIDC](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-protocols-oidc)
