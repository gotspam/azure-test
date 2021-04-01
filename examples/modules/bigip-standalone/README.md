
# Deploying BIG-IP Template

[![Releases](https://img.shields.io/github/release/f5networks/f5-azure-arm-templates-v2.svg)](https://github.com/f5networks/f5-azure-arm-templates-v2/releases)
[![Issues](https://img.shields.io/github/issues/f5networks/f5-azure-arm-templates-v2.svg)](https://github.com/f5networks/f5-azure-arm-templates-v2/issues)

## Contents

- [Deploying BIG-IP Template](#deploying-big-ip-template)
  - [Contents](#contents)
  - [Introduction](#introduction)
  - [Prerequisites](#prerequisites)
  - [Important Configuration Notes](#important-configuration-notes)
    - [Template Input Parameters](#template-input-parameters)
    - [Template Outputs](#template-outputs)

## Introduction

This ARM template creates a BIG-IP standalone instance, creates an optional application insight component, and optionally associates specified role definition with system assigned managed identity. Link this template to create BIG-IP VM(s) required for F5 deployments.

## Prerequisites

 - F5-bigip-runtime-init configuration file required. See https://github.com/F5Networks/f5-bigip-runtime-init for more details on F5-bigip-runtime-init SDK. See example runtime-init-conf.yaml in the repository.
 - Declarative Onboaring (DO) declaration: See example standalone_do_payg.json or standalone_do_bigiq.json in the repository.
 - AS3 declaration: See example standalone_a3.json in the repository.
 - Telemetry Streaming (TS) declaration if using custom metrics. See example standalone_ts.json in the repository.
 
## Important Configuration Notes

 - A sample template, 'sample_linked.json', has been included in this project. Use this example to see how to add bigip.json as a linked template into your templated solution.
- New vs existing Azure App Insights: When specifying a value for the appInsights input parameter, a new Azure App Insights resource is created. This resource is only available in regions that support the Azure Container Service; therefore, the App Insights and deploymentScript resources will be created in the West US 2 region.
- Troubleshooting: The log location for f5-bigip-runtime-init onboarding is ``/var/log/cloud/bigIpRuntimeInit.log``. By default, the log level is set to info; however, you can set a custom log level by exporting the F5_BIGIP_RUNTIME_INIT_LOG_LEVEL environment variable before invoking f5-bigip-runtime-init in commandToExecute: 
```export F5_BIGIP_RUNTIME_INIT_LOG_LEVEL=silly && bash ', variables('runtimeConfigPackage'), ' azure 2>&1```



### Template Input Parameters

| Parameter | Required | Description |
| --- | --- | --- |
| adminUsername | Yes | Enter a valid BIG-IP username. This creates the specified username on the BIG-IP with admin role. |
| appInsights | No | Valid values: empty value, an acceptable application insight component name. Creates application insight component with specified name. |
| customEmail | No | Enter an array of email addresses to be notified when scale up or scale down occurs. For example: ['myemail@email.com','myemail2@email.com']. |
| dnsLabel | Yes | Unique DNS Name for the Public IP address used to access the Virtual Machine and postfix resource names. |
| image | Yes |  There are two acceptable formats: Enter the URN of the image to use in Azure marketplace, or enter the ID of the custom image. An example URN value: 'f5-networks:f5-big-ip-byol:f5-big-ltm-2slot-byol:15.1.201000'. You can find the URNs of F5 marketplace images in the README for this template or by running the command: ``az vm image list --output yaml --publisher f5-networks --all``. See [this documentation](https://clouddocs.f5.com/cloud/public/v1/azure/Azure_download.html) for information on creating a custom BIG-IP image. |
| instanceType | Yes | Enter a valid instance type. |
| nsgId | No | Enter security group ID to use. Use default if you do not wish to apply an NSG policy. |
| publicIPAddressId | No | Public IP address resource ID for the BIG-IP instance. |
| roleDefinitionId | No | Enter a role definition ID you want to add to system managed identity. Leave default if system managed identity is not used. |
| runtimeConfig | Yes | Url to bigip-runtime-init configuration file or json string to use for configuration file. |
| sshKey | Yes | Supply the SSH public key you want to use to connect to the BIG-IP. |
| subnetId | Yes | Enter subnet ID to use. |
| tagValues | No | Default key/value resource tags will be added to the resources in this deployment, if you would like the values to be unique, adjust them as needed for each key. |
| useAvailabilityZones | Yes | This deployment can deploy resources into Azure Availability Zones (if the region supports it). If that is not desired, the input should be set 'No'. If the region does not support availability zones, the input should be set to 'No'.  |
| userAssignManagedIdentity | No | Enter user assigned management identity Id to be associated to vmss. Leave default if not used. |
| vmName| Yes | Name to use for Virtual Machine. |
| zoneChoice | Yes | If you want the VM placed in an Azure Availability Zone, and the Azure region you are deploying to supports it, specify the number of the existing Availability Zone you want to use. |

### Template Outputs

| Name | Description | Required Resource | Type |
| --- | --- | --- | --- |
| appInsightsComponentId| Application Insights resource ID | Application Insights | string |
| appInsightsInstrumentationKey | Application Insights Instrumentation Key | Application Insights | string |
| roleAssignmentId | Role Assignment resource ID | Role Definition | string |
| vmId| Virtual Machine resource ID | Virtual Machine | string |


**Declarative Onboarding Declaration Example**
- See https://clouddocs.f5.com/products/extensions/f5-declarative-onboarding/latest/ for more information.
- Onboarding declaration that supports payg onboarding.
- Note HOST_NAME has been defined in the example runtimeConfig. This is required to dynamically set each BIG-IP instance host name based on instance name set by Azure.

```json
{
    "Common": {
        "class": "Tenant",
        "dbvars": {
            "class": "DbVariables",
            "provision.extramb": 500,
            "restjavad.useextramb": true
        },
        "myDns": {
            "class": "DNS",
            "nameServers": [
                "8.8.8.8"
            ]
        },
        "myNtp": {
            "class": "NTP",
            "servers": [
                "0.pool.ntp.org"
            ],
            "timezone": "UTC"
        },
        "myProvisioning": {
            "asm": "nominal",
            "class": "Provision",
            "ltm": "nominal"
        },
        "mySystem": {
            "autoPhonehome": false,
            "class": "System",
            "hostname": "{{ HOST_NAME }}"
        }
    },
    "async": true,
    "class": "Device",
    "label": "my BIG-IP declaration for declarative onboarding",
    "schemaVersion": "1.0.0"
}
```

**Telemetry Streaming Declaration Example**
- See https://clouddocs.f5.com/products/extensions/f5-telemetry-streaming/latest/ for more information.
- Modify filter to match correct application insight name.

```json
{
    "Azure_Consumer": {
        "appInsightsResourceName": "dd-app-*",
        "class": "Telemetry_Consumer",
        "maxBatchIntervalMs": 5000,
        "maxBatchSize": 250,
        "trace": true,
        "type": "Azure_Application_Insights",
        "useManagedIdentity": true
    },
    "Bigip_Poller": {
        "actions": [
            {
                "includeData": {},
                "locations": {
                    "system": {
                        "cpu": true,
                        "networkInterfaces": {
                            "1.0": {
                                "counters.bitsIn": true
                            }
                        }
                    }
                }
            }
        ],
        "class": "Telemetry_System_Poller",
        "interval": 60
    },
    "class": "Telemetry"
}
```

**Application Services 3 (AS3) Declaration Example**
- See https://clouddocs.f5.com/products/extensions/f5-appsvcs-extension/latest/ for more information.
- AS3 declaration which supports WAF deployments.

```json
{
    "action": "deploy",
    "class": "AS3",
    "declaration": {
        "Sample_http_01": {
            "A1": {
                "My_ASM_Policy": {
                    "class": "WAF_Policy",
                    "ignoreChanges": true,
                    "url": "https://raw.githubusercontent.com/f5devcentral/f5-asm-policy-templates/master/generic_ready_template/Rapid_Depolyment_Policy_13_1.xml"
                },
                "class": "Application",
                "serviceMain": {
                    "class": "Service_HTTP",
                    "policyWAF": {
                        "use": "My_ASM_Policy"
                    },
                    "pool": "webPool",
                    "virtualAddresses": [
                        "0.0.0.0"
                    ],
                    "virtualPort": 80
                },
                "template": "http",
                "webPool": {
                    "class": "Pool",
                    "members": [
                        {
                            "serverAddresses": [
                                "10.0.0.2"
                            ],
                            "servicePort": 80
                        }
                    ],
                    "monitors": [
                        "http"
                    ]
                }
            },
            "class": "Tenant"
        },
        "class": "ADC",
        "label": "Sample 1",
        "remark": "HTTP with custom persistence",
        "schemaVersion": "3.0.0"
    },
    "persist": true
}
```
