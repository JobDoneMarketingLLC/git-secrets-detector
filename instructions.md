After saving the pre-commit file to your system, make it executable with:

chmod +x /path/to/pre-commit

For example, if you copy it to a repository’s hooks directory:

cp pre-commit /path/to/repo/.git/hooks/pre-commitchmod +x /path/to/repo/.git/hooks/pre-commit

For global installation:

mkdir -p ~/.git-hookscp pre-commit ~/.git-hooks/pre-commitchmod +x ~/.git-hooks/pre-commitgit config --global core.hooksPath ~/.git-hooks

Note: The executable bit is stored in the file’s permission metadata, not in the file content itself. Because of this, it must be explicitly set via chmod after the file is placed on your machine or server.
