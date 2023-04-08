## Managing git-credentials

### using ~/.git-credentials with PAT

Clone a repo using PAT
```bash
git clone https://<user_account>:<pat>@dev.azure.com/<account>/<project>/_git/<repo>

git clone https://<user_account>:<pat>@github.com/<repo>
```

Then a record will be stored in `~/.git-credentials`

### Using ssh keys

Using ssh keys for different orgs `~/.ssh/config`
```
Host github.com
  HostName github.com
  IdentityFile ~/.ssh/id_ed25519
  User git
  IdentitiesOnly yes
Host github-vivi
  HostName github.com
  IdentityFile ~/.ssh/id_vivi
  User git
  IdentitiesOnly yes
  

```