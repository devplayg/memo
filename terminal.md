# Terminal

### Profile

```bash
cat <<EOF > ~/.zshrc

alias .pro="vi ~/.zshrc"
alias pro=". ~/.zshrc"
alias ws=". ~/workspace"

export GOHOME=~/gohome
alias ll="ls -l"
EOF
```

gpg

```
brew install --cask gpg-suite
gpg --full-generate-key
4096
gpg --list-secret-keys --keyid-format LONG
gpg --armor --export D25CD2DCD5C372AD
```

profile

```
# os
alias .pro="vi ~/.zshrc"
alias pro=". ~/.zshrc"
alias ws="cd ~/workspace"
alias gh="cd ~/gohome"
alias ll="ls -l"

# docker
alias dps="docker ps -a"
alias dimg="docker images"
alias dnet="docker network ls"
alias ddown="docker-compose down"
alias dup="docker-compose up -d"

# db
alias db="docker exec -it db mysql -u root -pgrit devplayg"

# dev
alias dev="cd /Users/won/gohome/yuna-server/cmd/yuna"

```
