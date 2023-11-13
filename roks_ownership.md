When IBM user with admin account creates a ROKs cluster, that cluster is created with an apikey owned by the user.
If the user is removed from the account, the ROKS cluster may be stop working.

To make sure the ROKs cluster continues to work transfer the ownership of the cluster to a user that remains in the IBM account.
Steps to do this:

View current clusters:
```
ibmcloud ks clusters
```

View available resource groups:
```
ibmcloud resource groups
```

To view the current apikey owner for the cluster.
```
ibmcloud target -g <resource group that the cluster belongs to>
ibmcloud ks api-key info --cluster <cluster name or id>
```

Reset api key when user is removed
```
ibmcloud target -r <REGION>
ibmcloud target -g <RESOURCE_GROUP>
ibmcloud ks api-key reset --region <REGION>
ibmcloud ks cluster master refresh -c <cluster_name_or_ID>
```
