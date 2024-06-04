# devops-templates
Example how to use terraform plan and apply
Storage account for storing state of terraform is configured inside library variable group called app-env in below example it is going to be someapp-dev

```
trigger: 
  branches:
    include:
    - main
    - fix/*

resources:
  repositories:
    - repository: templates
      type: git
      name: AdamMachera/devops-templates
      ref: refs/heads/main
             

pool: 
  vmImage: ubuntu-latest


parameters:
- name: releaseFromFeatureBranch
  displayName: "Allow release from feature branch"
  type: boolean
  default: false
  values:
  - false
  - true

variables:
  - template: vars/global.yaml@templates  
  - name: app
    value: someapp
  - name: env
    value: dev
  - name: serviceConnection
    value: dev-connection

stages:
- template: stages/terraform-plan-template.yaml@templates
  parameters:
    dependsOn: []
    pool: ''
    name: ${{ variables.app }}
    env: ${{ variables.env }}
    terraformServiceConnection: ${{ variable.serviceConnection }}
    terraformDirectory: ./terraform 
    terraformEnvVariables:
      TF_VAR_app: ${{ variables.app }}
    releaseFromFeatureBranch: ${{ parameters.releaseFromFeatureBranch }}

- template: stages/terraform-apply-template.yaml@templates
  parameters:
    dependsOn: ['TerraformPlan_${{ variables.app }}_${{ variable.env }}']
    pool: ''
    name: ${{ variables.app }}
    env: ${{ variable.env }}
    terraformServiceConnection: ${{ variable.serviceConnection }}
    releaseFromFeatureBranch: ${{ parameters.releaseFromFeatureBranch }}
```