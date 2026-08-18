After uploading the pre-commit file to your server, make it executable with:
bash

chmod +x /path/to/pre-commit

For example, if you copy it to a repository’s hooks directory:
bash

cp pre-commit /path/to/repo/.git/hooks/pre-commit
chmod +x /path/to/repo/.git/hooks/pre-commit

For global installation:
bash

mkdir -p ~/.git-hooks
cp pre-commit ~/.git-hooks/pre-commit
chmod +x ~/.git-hooks/pre-commit
git config --global core.hooksPath ~/.git-hooks

The executable bit is stored in the file’s permission bits, not in the content, so it must be set on the server after the file is placed there.
