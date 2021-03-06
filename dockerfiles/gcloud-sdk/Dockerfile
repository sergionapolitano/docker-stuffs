FROM debian:buster-slim
MAINTAINER suryastef@gmail.com

# Set locale
ENV LANG C.UTF-8
ENV LC_ALL C.UTF-8
ENV LANGUAGE en_US.UTF-8

# Set terminal
ENV TERM xterm-256color

# Change Debian CLI to noninteractive important berfore running any command in debian based distro
ENV DEBIAN_FRONTEND noninteractive

# Hide apt-key warn
ENV APT_KEY_DONT_WARN_ON_DANGEROUS_USAGE=1

# install apt-utils to avoid warning
RUN apt-get update > /dev/null 2>&1 && apt-get upgrade -yq > /dev/null 2>&1 && apt-get install -yq apt-utils > /dev/null 2>&1

# Make sure apt-transport-https installed
RUN apt-get -yq install apt-transport-https ca-certificates gnupg --no-install-recommends > /dev/null 2>&1

# Install additional dependencies
RUN apt-get install -yq curl vim git zsh sudo watch unzip tmux > /dev/null 2>&1

# Add gcloud repository
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list

# Import the Google Cloud public key
RUN curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key --keyring /usr/share/keyrings/cloud.google.gpg add - > /dev/null 2>&1

# Update and install the Cloud SDK
RUN apt-get update > /dev/null 2>&1 && apt-get install -yq google-cloud-sdk > /dev/null 2>&1

# Add watchs
RUN echo "while true; do clear; date;echo;\$*;sleep 1; done" > /usr/local/bin/watchs
RUN chmod +x /usr/local/bin/watchs

# Clean apt cache
RUN apt-get clean > /dev/null 2>&1 && apt-get autoclean > /dev/null 2>&1

# Set working directory
WORKDIR /root

# Install Oh My ZSH
RUN zsh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended

# Disable Oh My ZSH auto update prompt
RUN sed -i '/DISABLE_UPDATE_PROMPT="true"/ s/#\s//' ~/.zshrc

# Install zsh-autosuggestion
RUN git clone -q --depth=1 https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# Install syntax highlighting
RUN git clone -q --depth=1 https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.zsh/zsh-syntax-highlighting
RUN echo "source ~/.zsh/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> ${ZDOTDIR:-$HOME}/.zshrc

# Add install powerlevel10k theme and enable additional plugins
COPY .p10k.zsh /root/.p10k.zsh
RUN git clone -q --depth=1 https://github.com/romkatv/powerlevel10k.git ~/.oh-my-zsh/themes/powerlevel10k
RUN sed -i -e 's/robbyrussell/powerlevel10k\/powerlevel10k/g' ~/.zshrc
RUN echo 'POWERLEVEL9K_DISABLE_CONFIGURATION_WIZARD=true' >> ~/.zshrc
RUN sed -i -e 's/(git)/(git kubectl gcloud terraform zsh-autosuggestions asdf helm tmux)/g' ~/.zshrc
RUN echo "[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh" >> .zshrc

# Install asdf and use specific kubectl & helm version for GKE
RUN git clone -q https://github.com/asdf-vm/asdf.git ~/.asdf && cd ~/.asdf && git checkout -q "$(git describe --abbrev=0 --tags)"
ENV PATH /root/.asdf/bin:$PATH
RUN asdf plugin-add kubectl > /dev/null 2>&1
RUN asdf install kubectl $(curl -sSL https://cloud.google.com/feeds/kubernetes-engine-stable-channel-release-notes.xml | grep -oE '[0-9]\.[0-9]+\.[0-9]+' | head -1)
RUN asdf global kubectl $(asdf list kubectl)
RUN asdf plugin-add helm
RUN asdf install helm latest > /dev/null 2>&1
RUN asdf global helm $(asdf list helm)
RUN asdf plugin-add terraform
RUN asdf install terraform latest > /dev/null 2>&1
RUN asdf global terraform $(asdf list terraform)

# Install oh my tmux
RUN curl -fsSLO https://raw.githubusercontent.com/gpakosz/.tmux/master/.tmux.conf.local
RUN curl -fsSLO https://raw.githubusercontent.com/gpakosz/.tmux/master/.tmux.conf
RUN sed -i '/set -g mouse on/ s/#//' ~/.tmux.conf.local
RUN sed -i '/tmux_conf_theme_left_separator_main=/ s/#//' ~/.tmux.conf.local
RUN sed -i '/tmux_conf_theme_left_separator_sub=/ s/#//' ~/.tmux.conf.local
RUN sed -i '/tmux_conf_theme_right_separator_main=/ s/#//' ~/.tmux.conf.local
RUN sed -i '/tmux_conf_theme_right_separator_sub=/ s/#//' ~/.tmux.conf.local
RUN sed -i 's/#ffff00,#ff00af,#00afff/#aaaa00,#00aa00,#00aaaa/g' ~/.tmux.conf.local
RUN sed -i 's/#000000,#e4e4e4,#e4e4e4/#000000,#000000,#e4e4e4/g' ~/.tmux.conf.local
RUN sed -i 's/#8a8a8a,#e4e4e4,#000000/#8a8a8a,#aaaaaa,#000000/g' ~/.tmux.conf.local
RUN sed -i 's/#080808,#d70000,#e4e4e4/#080808,#aa0000,#aaaaaa/g' ~/.tmux.conf.local
RUN sed -i '/set -gu prefix2/ s/#\s//' ~/.tmux.conf.local
RUN sed -i '/unbind C-a/ s/#\s//' ~/.tmux.conf.local
RUN echo 'unbind m' >> ~/.tmux.conf.local
RUN echo 'set -g default-terminal "screen-256color"' >> ~/.tmux.conf.local


# Install vim-airline
RUN git clone -q --depth=1 https://github.com/vim-airline/vim-airline ~/.vim/pack/dist/start/vim-airline
RUN git clone -q --depth=1 https://github.com/vim-airline/vim-airline-themes ~/.vim/pack/dist/start/vim-airline-themes
RUN git clone -q --depth=1 https://tpope.io/vim/fugitive.git ~/.vim/pack/tpope/start/fugitive
RUN curl -fsSLO https://raw.githubusercontent.com/suryastef/dotfiles/master/home/suryastef/.vimrc

# Install lsd
RUN dpkg -i $(curl -sw "%{filename_effective}" -LO $(curl -s https://api.github.com/repos/Peltoche/lsd/releases | grep browser_download_url | grep '64[.]deb' | head -n 1 | cut -d '"' -f 4))
RUN rm *.deb
RUN echo 'alias ls="lsd"' >> .zshrc

# Change default shell to ZSH
RUN chsh -s /bin/zsh root

# Revert Debian CLI to teletype important berfore running any command in debian based distro
ENV DEBIAN_FRONTEND teletype

# Custom volume for gcloud credential
VOLUME /root/.config/gcloud

# Default command to run when the container is running for the first time
CMD tmux