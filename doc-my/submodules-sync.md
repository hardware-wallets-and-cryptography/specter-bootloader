# Submodules Sync

## Troubleshooting: `Unable to find refs/remotes/origin/<branch> revision`

Running `git submodule update --remote` may fail with:

```
fatal: Unable to find refs/remotes/origin/<branch> revision in submodule path 'lib/secp256k1'
```

### Why it happens

[.gitmodules](../.gitmodules) points the submodules at our forks, each on its
configured branch. However, a submodule that was cloned earlier keeps the old
`origin` URL in its own git config (e.g. upstream `bitcoin-core/secp256k1`),
which has no such branch. Changing `.gitmodules` alone does not update that remote.

### Step-by-step fix

1. Check which remote the submodule currently uses:

   ```sh
   git -C lib/secp256k1 remote -v
   ```

   If it shows an upstream URL instead of our fork, continue.

2. Verify `.gitmodules` has the expected fork URL and branch:

   ```sh
   git config -f .gitmodules --get-regexp 'submodule\..*\.(url|branch)'
   ```

3. Copy the URLs from `.gitmodules` into each submodule's `origin`:

   ```sh
   git submodule sync
   ```

4. Make sure every submodule is initialized at the commit pinned by this repo
   (no version change):

   ```sh
   git submodule update --init --remote
   ```

5. Confirm the result:

   ```sh
   git submodule status
   ```

   ```sh
   git -C lib/secp256k1 remote -v   # should now show the fork
   ```

If `git status` shows the submodule pointers changed, review and commit them
only if you intend to bump the submodule versions.

> Tip: whenever a submodule URL changes in `.gitmodules`, run
> `git submodule sync` before `git submodule update`.
