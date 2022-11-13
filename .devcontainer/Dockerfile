FROM archlinux:latest

# Update and install packages
RUN pacman-key --init && pacman-key --populate archlinux
RUN pacman -Syu --noconfirm
RUN pacman -Sy --noconfirm coreutils base-devel zsh vim git openssh podman podman-docker
RUN rm -rf /etc/pacman.d/gnupg

# Change the shell to ZSH
RUN echo $(which zsh) >> /etc/shells
RUN chsh -s $(which zsh)
RUN echo 'autoload -Uz compinit promptinit' >> /root/.zshrc
RUN echo 'compinit' >> /root/.zshrc
RUN echo 'promptinit' >> /root/.zshrc
RUN echo 'export PS1="%B%F{cyan}%(4~|...|)%3~%F{white} %# %b%f%k"' >> /root/.zshrc
RUN echo 'alias ls="ls -al --color=auto"' >> /root/.zshrc
RUN echo 'alias mknew="npx mknew -s https://github.com/Shakeskeyboarde/templates.git"' >> /root/.zshrc
ENTRYPOINT [ "zsh" ]

# Install NodeJS
ENV NVM_DIR /root/.nvm
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.2/install.sh | bash
RUN . $NVM_DIR/nvm.sh \
    && nvm install 18 \
    && nvm alias default node \
    && nvm use node \
    && npm i -g npm@latest

# Configure GIT
RUN git config --global core.editor "vim"
RUN git config --global pull.rebase false
