[![Infrastructure Tests](https://www.bridgecrew.cloud/badges/github/rbenavente/shiftleftdemo/pci_dss_v321)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=rbenavente%2Fshiftleftdemo&benchmark=PCI-DSS+V3.2.1)
[![Infrastructure Tests](https://www.bridgecrew.cloud/badges/github/rbenavente/shiftleftdemo/general)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=rbenavente%2Fshiftleftdemo&benchmark=INFRASTRUCTURE+SECURITY)

[![Infrastructure Tests](https://www.bridgecrew.cloud/badges/github/rbenavente/shiftleftdemo/cis_kubernetes)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=rbenavente%2Fshiftleftdemo&benchmark=CIS+KUBERNETES+V1.5)


You can clone this repo and put it in the demo-build gogs server, so that can in turn be used in the demo-build Jenkins server with all pre-existing vars and passwors.
You can also point the demo-build server to this repo, once it becomes public, or to your fork, instead of copying to the gogs server.


PC_USER,PC_PASS and PC_CONSOLE are pre populated from the yaml file from the central demo build. 


These three items are default values in the yaml file and can be configured for the deployment:
  pc_token: 11111111-2222-3333-4444-555555555555
  pc_secret: 666666777777788888888999999=
  pc_api: us-west1.cloud.twistlock.com/us-3-12345678
