## Activation of the OCM Invitation Workflow in Nextcloud

##### Document version: v1.0.0
---
### Requirements

- Nextcloud Server - minumum versions: v33.0.9, v34.0.4, v35.0.0
- Contacts app - version: v8.9.0

### ! Prerequisite

- Install or upgrade to the Contacts app v8.9.0 **after** you've upgraded to the required Nextcloud Server version. <sup>[**Notes 1**](#notes)</sup>

### Activation

1. Activate the Contacts app: \
          `occ app:enable contacts`
2. Activate OCM invites: \
          `occ config:app:set --value 1 contacts ocm_invites_enabled`
3. Set the url of the mesh providers service: \
          `occ config:app:set --value {MESH_PROVIDERS_SERVICE_URL} contacts mesh_providers_service` \
The final `MESH_PROVIDERS_SERVICE_URL` is yet to be published. We have a test url ([here](https://ocm-invitation-workflow-mgmt-app.data.surf.nl/external-ocm-servers.json)) for you to test your installation. \
\
**\*** See [Available settings](#available-settings) for all available (optional) settings.

### Post activation checks

- To test the WAYF page use this url: \
            `https://{your_provider_fqdn}/apps/contacts/wayf?token=fake-token` \
  This should display the WAYF page.
- The WAYF page loads the mesh providers from cache. The cache is filled by background `cron` jobs which consult the mesh providers service. There are actually 2 jobs: one that completely refreshes the cache every 24 hours, and one that updates the cache by checking for new providers every 15 min. \
  You may check the contents of the cache with the following `occ` command: \
            `occ config:app:get contacts federations_cache` \
  Should this display no providers, the background jobs may not have run yet. In that case you can trigger the background jobs manually. \
  List the jobs:  \
            `occ background-job:list` \
            (look for `OCA\Contacts\Cron\UpdateOcmProviders` or `OCA\Contacts\Cron\UpdateDeltaOcmProviders`) \
  And run one of both: \
            `occ background-job:execute {id}` \
  And check the cache contents again.
- When a provider that should be, but is not, listed on the WAYF page, it may be that it is not discoverable. \
  To verify that a provider is discoverable, navigate to:  \
            `https://{fqdn_provider}/.well-knowm/ocm`

---

### Available settings

| Config key                        | Default value  |                                                                                                            |
|-----------------------------------|----------------|------------------------------------------------------------------------------------------------------------|
| `ocm_invites_enabled`             | `false`        | Turn OCM invites feature on/off (default - off)                                                      |
| `ocm_invites_optional_mail`       | `false`        | Make sending an invitation email optional (default - do send invitation email)                       |
| `ocm_invites_encoded_copy_button` | `false`        | Do (not) display the encoded invite copy button of an existing invitation (default - do not display) |
| `mesh_providers_service`          | *empty string* | The mesh providers service url (default - not set)                                                   |
| `wayf_endpoint`                   | *empty string* | The endpoint base of the wayf page (without parameters) (default - not set)                          |
| `ocm_invites_disable_ssrf_guard`  | `false`        | **Do not activate**; for development purposes only (default - not disabled)                              |

---

#### Notes:

1. A bug has been discovered that potentially removes the invitations table previously created by the Contacts app when the server is upgraded to the required version. This bug will be fixed in the upcoming updates for each of the stable releases (33, 34, 35). \
To deal with this bug, at this moment it is highly recommended to install the required version of the Contacts app after! the server has been upgraded to one of its required versions.