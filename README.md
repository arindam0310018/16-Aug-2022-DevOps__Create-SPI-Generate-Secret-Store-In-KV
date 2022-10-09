# CREATE SERVICE PRINCIPAL AND STORE SECRET IN KEY VAULT USING AZ DEVOPS

Greetings to my fellow Technology Advocates and Specialists.

In this Session, I will demonstrate __how to Create Service Principal and Store Secret in Key Vault Using Azure DevOps.__

I had the Privilege to talk on this topic in __TWO__ Azure Communities:-

| __NAME OF THE AZURE COMMUNITY__ | __TYPE OF SPEAKER SESSION__ |
| --------- | --------- |
| __Microsoft Azure Bern User Group__ | __In Person__ |
| __Journey to the Cloud 9.0__ | __Virtual__ |


| __IN-PERSON SESSION:-__ |
| --------- |
| I presented this Demo as a part of __AZURE DEVOPS: TAKEAWAYS BEST PRACTISES AND LIVE DEMOS__ In-Person Speaker Session in __MICROSOFT AZURE BERN USER GROUP__ Forum/Platform. |
| __Event Meetup Announcement:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/cpz6voitr8v43psbgsii.jpg) |
| __Moment Captured with Founders of MICROSOFT AZURE BERN USER GROUP "STEFAN JOHNER", "STEFAN ROTH", "PAUL AFFENTRANGER" and Co-organizer "DAMIEN BOWDEN":-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jxz1bp0dw976e7s3j3c0.JPG) |
| __VIRTUAL SESSION:-__ |
| __LIVE DEMO__ was Recorded as part of my Presentation in __JOURNEY TO THE CLOUD 9.0__ Forum/Platform |
| Duration of My Demo = __55 Mins 42 Secs__ |
| [![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/EGIOzEpOxzE/0.jpg)](https://www.youtube.com/watch?v=EGIOzEpOxzE) |


| __USE CASE:-__ |
| --------- |
| Cloud Engineer __DOES NOT__ have access to __Azure Active Directory (AAD)__ to Create Service Principal. |
| Cloud Engineer __CANNOT ELEVATE__ rights using __PIM (Privileged Identity Management)__ to Create Service Principal. |


| __AUTOMATION OBJECTIVE:-__ |
| --------- |
| Validate If the Service Principal Exists. If __Yes__, Pipeline will __FAIL__. |
| Validate If Resource Group Containing Key Vault Exists. If __No Resource Group Found__, Pipeline will __FAIL__. |
| Validate If Key Vault Exists inside the Specified Resource Group. If __No Key Vault Found__, Pipeline will __FAIL__. |
| If All of the above validation is __SUCCESSFUL__, Pipeline will then Create the Service Principal, Generate Secret and Store it in the Key Vault. |


| __IMPORTANT NOTE:-__ |
| --------- |
The YAML Pipeline is tested on __WINDOWS BUILD AGENT__ Only!!!


| __REQUIREMENTS:-__ |
| --------- |

1. Azure Subscription.
2. Azure DevOps Organisation and Project.
3. Service Principal with Required RBAC ( __Contributor__) applied on Subscription or Resource Group(s).
4. Azure Resource Manager Service Connection in Azure DevOps.


| __HOW DOES MY CODE PLACEHOLDER LOOKS LIKE:-__ |
| --------- |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8w382h4710obhp8h113r.png) |


| PIPELINE CODE SNIPPET:- | 
| --------- |

| AZURE DEVOPS YAML PIPELINE (azure-pipelines-spi-validate-create-store-in-KV-v1.0.yml):- | 
| --------- |

```
trigger:
  none

######################
#DECLARE PARAMETERS:-
######################
parameters:
- name: SubscriptionID
  displayName: Subscription ID Details Follow Below:-
  type: string
  default: 210e66cb-55cf-424e-8daa-6cad804ab604
  values:
  - 210e66cb-55cf-424e-8daa-6cad804ab604

- name: RGNAME
  displayName: Please Provide the Resource Group Name:-
  type: object
  default: 

- name: KVNAME
  displayName: Please Provide the Keyvault Name:-
  type: object
  default: 

- name: SPINAME
  displayName: Please Provide the Service Principal Name:-
  type: object
  default:

######################
#DECLARE VARIABLES:-
######################
variables:
  ServiceConnection: amcloud-cicd-service-connection
  BuildAgent: windows-latest

#########################
# Declare Build Agents:-
#########################
pool:
  vmImage: $(BuildAgent)

###################
# Declare Stages:-
###################

stages:

- stage: CREATE_SERVICE_PRINCIPAL 
  jobs:
  - job: CREATE_SERVICE_PRINCIPAL 
    displayName: CREATE SERVICE PRINCIPAL
    steps:
    - task: AzureCLI@2
      displayName: VALIDATE AND CREATE SPI
      inputs:
        azureSubscription: $(ServiceConnection)
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |
          az --version
          az account set --subscription ${{ parameters.SubscriptionID }}
          az account show
          $i = az ad sp list --display-name ${{ parameters.SPINAME }} --query [].appDisplayName -o tsv
          if ($i -ne "${{ parameters.SPINAME }}") {
            $j = az group exists -n ${{ parameters.RGNAME }}
                if ($j -eq "true") {
                  $k = az keyvault list --resource-group ${{ parameters.RGNAME }} --query [].name -o tsv		
                      if ($k -eq "${{ parameters.KVNAME }}") {
                        $spipasswd = az ad sp create-for-rbac -n ${{ parameters.SPINAME }} --query "password" -o tsv
                        az keyvault secret set --name ${{ parameters.SPINAME }} --vault-name ${{ parameters.KVNAME }} --value $spipasswd
                        echo "##################################################################"
                        echo "Service Principal ${{ parameters.SPINAME }} created successfully and the Secret Stored inside Key Vault ${{ parameters.KVNAME }} in the Resource Group ${{ parameters.RGNAME }}!!!"
                        echo "##################################################################"
                        }				
                      else {
                      echo "##################################################################"
                      echo "Key Vault ${{ parameters.KVNAME }} DOES NOT EXISTS in Resource Group ${{ parameters.RGNAME }}!!!"
                      echo "##################################################################"
                      exit 1
                          }
                }
                else {
                echo "##################################################################"
                echo "Resource Group ${{ parameters.RGNAME }} DOES NOT EXISTS!!!"
                echo "##################################################################"
                exit 1
                    }
          }
          else {
          echo "##################################################################"
          echo "Service Principal ${{ parameters.SPINAME }} EXISTS and hence Cannot Proceed with Deployment!!!"
          echo "##################################################################"
          exit 1
              }

```

__Now, let me explain each part of YAML Pipeline for better understanding.__

| __PART #1:-__ | 
| --------- |

| __BELOW FOLLOWS PIPELINE RUNTIME VARIABLES CODE SNIPPET:-__ | 
| --------- |

```
######################
#DECLARE PARAMETERS:-
######################
parameters:
- name: SubscriptionID
  displayName: Subscription ID Details Follow Below:-
  type: string
  default: 210e66cb-55cf-424e-8daa-6cad804ab604
  values:
  - 210e66cb-55cf-424e-8daa-6cad804ab604

- name: RGNAME
  displayName: Please Provide the Resource Group Name:-
  type: object
  default: 

- name: KVNAME
  displayName: Please Provide the Keyvault Name:-
  type: object
  default: 

- name: SPINAME
  displayName: Please Provide the Service Principal Name:-
  type: object
  default:

```

| __PART #2:-__ | 
| --------- |

| __BELOW FOLLOWS PIPELINE VARIABLES CODE SNIPPET:-__ | 
| --------- |

```
######################
#DECLARE VARIABLES:-
######################
variables:
  ServiceConnection: amcloud-cicd-service-connection
  BuildAgent: windows-latest

```

| __NOTE:-__ | 
| --------- |
| Please change the values of the variables accordingly. |
| The entire YAML pipeline is build using __Runtime Parameters and Variables__. No Values are Hardcoded. |


| __PART #3:-__ | 
| --------- |

| __BELOW FOLLOWS THE CONDITIONS AND LOGIC DEFINED IN THE PIPELINE (AS MENTIONED ABOVE IN THE "AUTOMATION OBJECTIVE"):-__ | 
| --------- |

```
inlineScript: |
          az --version
          az account set --subscription ${{ parameters.SubscriptionID }}
          az account show
          $i = az ad sp list --display-name ${{ parameters.SPINAME }} --query [].appDisplayName -o tsv
          if ($i -ne "${{ parameters.SPINAME }}") {
            $j = az group exists -n ${{ parameters.RGNAME }}
                if ($j -eq "true") {
                  $k = az keyvault list --resource-group ${{ parameters.RGNAME }} --query [].name -o tsv		
                      if ($k -eq "${{ parameters.KVNAME }}") {
                        $spipasswd = az ad sp create-for-rbac -n ${{ parameters.SPINAME }} --query "password" -o tsv
                        az keyvault secret set --name ${{ parameters.SPINAME }} --vault-name ${{ parameters.KVNAME }} --value $spipasswd
                        echo "##################################################################"
                        echo "Service Principal ${{ parameters.SPINAME }} created successfully and the Secret Stored inside Key Vault ${{ parameters.KVNAME }} in the Resource Group ${{ parameters.RGNAME }}!!!"
                        echo "##################################################################"
                        }				
                      else {
                      echo "##################################################################"
                      echo "Key Vault ${{ parameters.KVNAME }} DOES NOT EXISTS in Resource Group ${{ parameters.RGNAME }}!!!"
                      echo "##################################################################"
                      exit 1
                          }
                }
                else {
                echo "##################################################################"
                echo "Resource Group ${{ parameters.RGNAME }} DOES NOT EXISTS!!!"
                echo "##################################################################"
                exit 1
                    }
          }
          else {
          echo "##################################################################"
          echo "Service Principal ${{ parameters.SPINAME }} EXISTS and hence Cannot Proceed with Deployment!!!"
          echo "##################################################################"
          exit 1
              }

```

__NOW ITS TIME TO TEST !!!...__

| __TEST CASES:-__ | 
| --------- |

| __TEST CASE #1: SERVICE PRINCIPAL NAME DOES NOT EXISTS, RESOURCE GROUP AND KEY VAULT EXISTS:-__ | 
| --------- |
| __DESIRED OUTPUT: PIPELINE EXECUTED SUCCESSFULLY. SERVICE PRINCIPAL GOT CREATED. SECRET WAS GENERATED AND STORED IN KEY VAULT.__ |
| __PIPELINE RUNTIME VARIABLES VALUE:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2wa0r9ldwdlzmedhp8f8.JPG) |
| __PIPELINE EXECUTED SUCCESSFULLY:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2uwrg4tb2kp1jnlszz0o.png) |
| __SERVICE PRINCIPAL CREATED + SECRET GENERATED:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/znfupjv7m1zb1nbprmt5.JPG) |
| __SECRET STORED IN KEY VAULT:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pgbzbchnml0gvd7roopg.JPG) |
| __TEST CASE #2: SERVICE PRINCIPAL, RESOURCE GROUP AND KEY VAULT EXISTS:-__ |
| __DESIRED OUTPUT: PIPELINE FAILS STATING THAT THE SERVICE PRINCIPAL ALREADY EXISTS.__ |
| __PIPELINE FAILED:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/o1js4e2nakdfdg87tlvt.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/j3qhoe0vai90f7mupjdc.png) |
| __TEST CASE #3: KEYVAULT EXISTS BUT SERVICE PRINCIPAL AND RESOURCE GROUP DOES NOT EXISTS:-__ |
| __DESIRED OUTPUT: PIPELINE FAILS STATING THAT THE RESOURCE GROUP DOES NOT EXISTS.__ |
| __PIPELINE FAILED:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2jlovht9zcxw5o9nczv2.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8az53s0snklg28d4w2uk.png) |
| __TEST CASE #4: RESOURCE GROUP EXISTS BUT SERVICE PRINCIPAL AND KEY VAULT DOES NOT EXISTS:-__ |
| __DESIRED OUTPUT: PIPELINE FAILS STATING THAT THE KEY VAULT DOES NOT EXISTS.__ |
| __PIPELINE FAILED:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/n40nu216mc3pvc49ayo5.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ggaetc84r2kbltlb534u.png) |

__Hope You Enjoyed the Session!!!__

__Stay Safe | Keep Learning | Spread Knowledge__
