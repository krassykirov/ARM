{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "environment": {
      "type": "string"
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location (region) for all resources."
      }
    },
    "appServiceSku": {
      "type": "string",
      "defaultValue": "B1",
      "metadata": {
        "description": "The SKU of App Service Plan "
      }
    },
    "dockerImageName": {
      "type": "string",
      "defaultValue": "krassy.azurecr.io/krmyfunction:latest"
    },
    "dockerRegistryUrl": {
      "type": "string",
      "defaultValue": "https://krassy.azurecr.io"
    },
    "dockerRegistryUsername": {
      "type": "string",
      "defaultValue": "krassy"
    },
    "dockerRegistryPassword": {
      "type": "securestring",
      "defaultValue": ""
    }
  },
 
  "variables": {
    "name": "kr",
    "webAppPortalName": "[concat(variables('name'), parameters('environment'), '-webapp')]",
    "appServicePlanName": "[concat(variables('name'), parameters('environment'),'-hosting')]"
  },
  "resources": [       
    {
      "apiVersion": "2017-08-01",
      "type": "Microsoft.Web/serverfarms",
      "kind": "linux",
      "name": "[variables('appServicePlanName')]",
      "location": "[parameters('location')]",
      "comments": "This app service plan is used for the web app and slots.",
      "properties": {
        "reserved": true
      },
      "dependsOn": [],
      "sku": {
        "name": "[parameters('appServiceSku')]"
      }
    },
    {
      "apiVersion": "2016-08-01",
      "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('appServicePlanName'))]"
      ],
      "kind": "app,linux,container",
      "location": "[parameters('location')]",
      "name": "[variables('webAppPortalName')]",
      "properties": {
        "name": "[variables('webAppPortalName')]",
        "siteConfig": {
          "alwaysOn": true,
          "appSettings": [
            {
              "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
              "value": "false"
            },
            {
              "name": "DOCKER_REGISTRY_SERVER_URL",
              "value": "[parameters('dockerRegistryUrl')]"
            },
            {
              "name": "DOCKER_REGISTRY_SERVER_USERNAME",
              "value": "[parameters('dockerRegistryUsername')]"
            },
            {
              "name": "DOCKER_REGISTRY_SERVER_PASSWORD",
              "value": "[parameters('dockerRegistryPassword')]"
            }
          ],
          "linuxFxVersion": "[concat('DOCKER|', parameters('dockerImageName'))]"
        },
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('appServicePlanName'))]"
      },
      "type": "Microsoft.Web/sites"
    }
  ]
  }
