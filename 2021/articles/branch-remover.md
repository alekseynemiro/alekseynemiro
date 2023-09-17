# Easy way to remove branches from GitHub repos

> Large projects can accumulate a large number of unnecessary branches. In general, there is nothing wrong with that. But in general, there is nothing good either, because merged branches will, most likely, never be needed again. And in the end, these useless branches can complicate navigation and create confusion.

> [!WARNING]
> **Disclaimer: Using this tool, you are aware that you may lose important data and fully accept responsibility for this. Please read the documentation carefully before using this tool in a real-world environment.**

To quickly remove unnecessary branches on **GitHub**, you can use a small utility called branch-remover. You can install **branch-remover** via **Node Package Manager**:

```
npm install --global branch-remover
```

To remove branches in a repository, you will need to specify the repository name, owner name, access token and remove rules.

You can find the owner and repository name in the project URL. For example:

```
https://github.com/sfm-tools/branch-remover
                   ^^^^^^^^  ^^^^^^^^^^^^^^
                   owner     repo
```

And an access token you can get in the your profile settings. Check the following link for details: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token

![Creating an access token on the GitHub](assets/AccessToken.webp)

Use the owner name (`%owner%`), repository name (`%repo%`) and your access token (`%token%`) to remove all merged branches in the repository:

```
branch-remover --github.owner %owner% --github.repo %repo% --github.token %token%
```

For example, remove all merged branches from https://github.com/sfm-tools/branch-remover:

```
branch-remover --github.owner sfm-tools --github.repo branch-remover --github.token ghp_eeSoSlAbigHUtRelABELaaBbErAvDswoOrSd
```

The program will check all the branches in the repository and when it finds a merged branch, it will offer to remove it:

![The result of the program](assets/RemoveResults.webp)

By default, the program will ask to remove every merged branch, and the default answer is `N` (no). This may not be very convenient when you are 100% sure that you really need to remove all matching branches. Add the **yes** flag so that the default answer is always `Y` (yes):

```
branch-remover --github.owner %owner% --github.repo %repo% --github.token %token% --yes
```

Use the `quiet` flag so that the program does not ask for confirmation to remove found branches:

> [!WARNING]
> WARNING: Be extremely careful when using quiet mode! This can lead to data loss!

```
branch-remover --github.owner %owner% --github.repo %repo% --github.token %token% --quiet
```

**Use the `test` parameter to safely test how the program will work:**

```
branch-remover --github.owner %owner% --github.repo %repo% --github.token %token% --quiet --test
```

![Test mode](assets/TestMode.webp)

---

If necessary, you can configure the rules for removing branches. For example, using the `merged` parameter, you can specify how long a branch should be merged so the program can remove the branch. The following example shows the removing of branches that were merged no earlier than 2 months ago:

```
branch-remover --github.owner %owner% --github.repo %repo% --github.token %token% --merged "2 months"
```

The program can remove not only merged branches, but also non-merged ones too. Use the `stale` parameter to specify the time elapsed since the last update, after which branches can be removed. The following example shows the removing of all branches that were last updated a year ago:

```
branch-remover --github.owner %owner% --github.repo %repo% --github.token %token% --stale "1 year"
```

You can also exclude certain branches by using the `ignore` parameter:

```
branch-remover --github.owner %owner% --github.repo %repo% --github.token %token% --ignore "^(master|main|dev|beta)$"
```

---

You can use **branch-remover** to manually remove branches from your GitHub repositories. Or you can automate the process using the system task manager. Whichever method you choose, always test your configuration with the `test` flag.

---
Aleksey Nemiro  
2021-10-03

https://medium.com/git-hero/easy-way-to-remove-branches-from-github-repos-1cbc777d9140
