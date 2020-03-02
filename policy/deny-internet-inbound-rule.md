# Deny creating a VM with internet inbound port

### Deny creating NSG with internet inbound rules

```sh
# Microsoft.Network/networkSecurityGroups
# https://github.com/Azure/Community-Policy/blob/master/Policies/Network/block-nsg-creations-updates/azurepolicy.rules.json
az policy definition create --name deny-nsg-internet-inbound --mode All --rules @nsg-policy.json
az policy assignment create --name deny-nsg-internet-inbound-assignment --scope /subscriptions/0b1f6471-1bf0-4dda-aec3-cb9272f09590 --policy deny-nsg-internet-inbound

# Test
az resource create --id "/subscriptions/0b1f6471-1bf0-4dda-aec3-cb9272f09590/resourceGroups/{}/providers/Microsoft.Network/networkSecurityGroups/{}" --location eastasia --properties @nsg.json
```

Files: [nsg-policy.json](nsg-policy.json), [nsg.json](nsg.json)

### Deny creating internet inbound rule on existing NSG

```sh
# Microsoft.Network/networkSecurityGroups/securityRules
# https://github.com/Azure/azure-policy/blob/master/samples/Network/deny-nsg-inbound-allow-all/azurepolicy.json
az policy definition create --name deny-nsg-rule-internet-inbound --mode All --rules @nsg-rule-policy.json
az policy assignment create --name deny-nsg-rule-internet-inbound-assignment --scope /subscriptions/0b1f6471-1bf0-4dda-aec3-cb9272f09590 --policy deny-nsg-rule-internet-inbound

# Test
az network nsg rule create -g {} --nsg-name {} --name allow-all --priority 100 --direction Inbound --source-address-prefixes "*" --destination-port-ranges 22
```

Files: [nsg-rule-policy.json](nsg-rule-policy.json)

### Deny creating NIC without NSG property

```sh
# Microsoft.Network/networkInterfaces
# Deny creating NIC without NSG property
az policy definition create --name deny-nic-no-nsg --mode All --rules @nic-policy.json
az policy assignment create --name deny-nic-no-nsg-assignment --scope /subscriptions/0b1f6471-1bf0-4dda-aec3-cb9272f09590 --policy deny-nic-no-nsg

# Test
az network nic update --network-security-group "" --ids /subscriptions/0b1f6471-1bf0-4dda-aec3-cb9272f09590/resourceGroups/{}/providers/Microsoft.Network/networkInterfaces/{}
```

Files: [nic-policy.json](nic-policy.json)

### Trigger policy evaluation

To manually trigger policy evaluation, use **Policy States - Trigger Subscription Evaluation** API: https://docs.microsoft.com/en-us/rest/api/policy-insights/policystates/triggersubscriptionevaluation

```
az rest --method post --uri https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/triggerEvaluation?api-version=2019-10-01
```