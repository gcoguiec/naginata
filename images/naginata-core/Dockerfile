ARG naginata_url=https://github.com/gcoguiec/naginata
ARG naginata_authors=Naginata Community
ARG naginata_vendor=Naginata
ARG naginata_user=samurai
ARG naginata_uid=1000

ARG build_platform
ARG build_revision
ARG build_version
ARG build_created_at

ARG helix_dir=/usr/lib/helix

FROM alpine:3.20.3 AS stage1
ARG build_platform
ARG helix_dir
ARG grc_version=1.13
ARG grc_checksum=a7b10d4316b59ca50f6b749f1d080cea0b41cb3b7258099c3eb195659d1f144f
ARG helix_version=24.07
ARG helix_checksum=0f466ed2de039a7eca6faf29fc0db712c92e1a59d0bdc7e8916c717ceee8b3b3
ARG fzf_version=0.55.0
ARG fzf_checksum=805383f71bca7f8fb271ecd716852aea88fd898d5027d58add9e43df6ea766da
ARG bottom_version=0.10.2
ARG bottom_checksum=1db45fe9bc1fabb62d67bf8a1ea50c96e78ff4d2a5e25bf8ae8880e3ad5af80a
ARG zoxide_version=0.9.4
ARG zoxide_checksum=ec002bdca37917130ae34e733eb29d4baa03b130c4b11456d630a01a938e0187
ARG eza_version=0.19.2
ARG eza_checksum=db4897ef7f58d0802620180e0b13bb35563e03c9de66624206b35dcad21007f8
ARG ripgrep_version=14.1.1
ARG ripgrep_checksum=4dad02a2f9c8c3c8d89434e47337aa654cb0e2aa50e806589132f186bf5c2b66
ARG dust_version=1.1.1
ARG dust_checksum=98cae3e4b32514e51fcc1ed07fdbe6929d4b80942925348cc6e57b308d9c4cb0
ARG yazi_version=0.3.3
ARG yazi_checksum=fe2a458808334fe20eff1ab0145c78d684d8736c9715e4c51bce54038607dc4e
ARG bat_version=0.24.0
ARG bat_checksum=907554a9eff239f256ee8fe05a922aad84febe4fe10a499def72a4557e9eedfb
ARG xh_version=0.22.2
ARG xh_checksum=32a6470ab705aba4c37fce9806202dcc0ed24f55e091e2f4bdf7583108a3da63
ARG atac_version=0.18.0
ARG atac_checksum=509984a38f5609210ee824f8f9675f869d3ae50a51ad8204351e2065711257b0
ARG fish_version=3.7.1
ARG fish_checksum=a3ea79c8affb9fa3c510ec6f12b9a8d176fec2d532ea7c41dfd4da4205272f19
ARG zellij_version=0.40.1
ARG zellij_checksum=1f0bfa13f2dbe657d76341a196f98a3b4caa47ac63abee06b39883a11ca220a8

ENV REQUESTS_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt
ENV HELIX_DISABLE_AUTO_GRAMMAR_BUILD=1
ENV FZF_VERSION="${fzf_version}"
ENV FZF_REVISION="naginata"

WORKDIR /tmp

RUN apk update -q \
  && apk add --upgrade --no-cache ca-certificates-bundle cmake make gcc g++ \
    musl-dev go bash git perl upx ncurses-dev pcre2-dev samurai

# grc
RUN wget -q https://github.com/garabik/grc/archive/refs/tags/v${grc_version}.tar.gz -O grc.tar.gz \
  && echo "${grc_checksum} grc.tar.gz" | sha256sum -c - \
  && mkdir /usr/share/grc grc \
  && tar xf grc.tar.gz --strip-components=1 -C grc \
  && cp grc/colourfiles/conf.* /usr/share/grc \
  && cp grc/grc.conf /etc \
  && cp grc/grc grc/grcat /usr/bin/

# rustup
RUN wget -q https://sh.rustup.rs -O install-rustup.sh \
  && sh install-rustup.sh -q -y --default-toolchain nightly \
  && rm install-rustup.sh \
  && source "$HOME/.cargo/env" \
  && rustup component add rust-src --toolchain nightly-${build_platform}-unknown-linux-musl

# helix
RUN wget -q https://github.com/helix-editor/helix/archive/refs/tags/${helix_version}.tar.gz -O helix.tar.gz \
  && echo "${helix_checksum} helix.tar.gz" | sha256sum -c - \
  && mkdir helix \
  && tar xf helix.tar.gz --strip-component=1 -C helix \
  && cd helix \
  && source "$HOME/.cargo/env" \
  && RUSTFLAGS="-Zfmt-debug=none -Zlocation-detail=none -C target-feature=-crt-static" cargo +nightly build \
    --config 'profile.release.strip = true' \
    --config 'profile.release.lto = "fat"' \
    --config 'profile.release.panic = "abort"' \
    --config 'profile.release.opt-level = "z"' \
    -Z build-std=std,panic_abort \
    -Z build-std-features="optimize_for_size" \
    --target ${build_platform}-unknown-linux-musl \
    --release \
    --locked \
  && mkdir -pv ${helix_dir} \
  && cp -r runtime ${helix_dir} \
  && upx -q --best --lzma --overlay=strip target/${build_platform}-unknown-linux-musl/release/hx \
  && cp target/${build_platform}-unknown-linux-musl/release/hx /usr/bin/ \
  && hx --health

# fzf
RUN wget -q https://github.com/junegunn/fzf/archive/refs/tags/v${fzf_version}.tar.gz -O fzf.tar.gz \
  && echo "${fzf_checksum} fzf.tar.gz" | sha256sum -c - \
  && mkdir fzf \
  && tar xf fzf.tar.gz --strip-component=1 -C fzf \
  && cd fzf \
  && make \
  && make install \
  && upx -q --best --ultra-brute bin/fzf \
  && cp bin/fzf /usr/bin/ \
  && fzf --version

# bottom
RUN wget -q https://github.com/ClementTsang/bottom/archive/refs/tags/${bottom_version}.tar.gz -O btm.tar.gz \
  && echo "${bottom_checksum} btm.tar.gz" | sha256sum -c - \
  && mkdir btm \
  && tar xf btm.tar.gz --strip-component=1 -C btm \
  && cd btm \
  && source "$HOME/.cargo/env" \
  && RUSTFLAGS="-Zfmt-debug=none -Zlocation-detail=none -Ctarget-cpu=native -Ctarget-feature=-crt-static" cargo build \
    --config 'profile.release.strip = true' \
    --config 'profile.release.opt-level = "z"' \
    --config 'profile.release.panic = "abort"' \
    --target ${build_platform}-unknown-linux-musl \
    --release \
    --locked \
  && upx -q --best --lzma --overlay=strip target/${build_platform}-unknown-linux-musl/release/btm \
  && cp target/${build_platform}-unknown-linux-musl/release/btm /usr/bin/ \
  && btm --version

# zoxide
RUN wget -q https://github.com/ajeetdsouza/zoxide/archive/refs/tags/v${zoxide_version}.tar.gz -O zoxide.tar.gz \
  && echo "${zoxide_checksum} zoxide.tar.gz" | sha256sum -c - \
  && mkdir zoxide \
  && tar xf zoxide.tar.gz --strip-component=1 -C zoxide \
  && cd zoxide \
  && source "$HOME/.cargo/env" \
  && RUSTFLAGS='-Zfmt-debug=none -Zlocation-detail=none -Ctarget-feature=-crt-static' cargo build \
      --config 'profile.release.opt-level = "z"' \
      --config 'profile.release.panic = "abort"' \
      -Z build-std=std,panic_abort \
      -Z build-std-features="optimize_for_size" \
      --target ${build_platform}-unknown-linux-musl \
      --release \
      --locked \
  && upx -q --best --lzma --overlay=strip target/${build_platform}-unknown-linux-musl/release/zoxide \
  && cp target/${build_platform}-unknown-linux-musl/release/zoxide /usr/bin/ \
  && zoxide --version

# eza
RUN wget -q https://github.com/eza-community/eza/archive/refs/tags/v${eza_version}.tar.gz -O eza.tar.gz \
  && echo "${eza_checksum} eza.tar.gz" | sha256sum -c - \
  && mkdir eza \
  && tar xf eza.tar.gz --strip-component=1 -C eza \
  && cd eza \
  && source "$HOME/.cargo/env" \
  && RUSTFLAGS='-Zfmt-debug=none -Zlocation-detail=none' cargo +nightly build \
      --config 'profile.release.opt-level = "z"' \
      --target ${build_platform}-unknown-linux-musl \
      --release \
      --no-default-features \
      --locked \
  && upx -q --best --lzma --overlay=strip target/${build_platform}-unknown-linux-musl/release/eza \
  && cp target/${build_platform}-unknown-linux-musl/release/eza /usr/bin/ \
  && eza --version

# ripgrep
RUN wget -q https://github.com/BurntSushi/ripgrep/archive/refs/tags/${ripgrep_version}.tar.gz -O rg.tar.gz \
  && echo "${ripgrep_checksum} rg.tar.gz" | sha256sum -c - \
  && mkdir rg \
  && tar xf rg.tar.gz --strip-component=1 -C rg \
  && cd rg \
  && source "$HOME/.cargo/env" \
  && RUSTFLAGS='-Zfmt-debug=none -Zlocation-detail=none -Ctarget-feature=-crt-static' \
    cargo build --config 'profile.release.opt-level = "z"' \
      --config 'profile.release.strip = true' \
      -Z build-std=std,panic_abort \
      -Z build-std-features='optimize_for_size' \
      --target ${build_platform}-unknown-linux-musl \
      --release \
      --locked \
  && upx -q --best --lzma --overlay=strip target/${build_platform}-unknown-linux-musl/release/rg \
  && cp target/${build_platform}-unknown-linux-musl/release/rg /usr/bin/ \
  && rg --version

# dust
RUN wget -q https://github.com/bootandy/dust/archive/refs/tags/v${dust_version}.tar.gz -O dust.tar.gz \
  && echo "${dust_checksum} dust.tar.gz" | sha256sum -c - \
  && mkdir dust \
  && tar xf dust.tar.gz --strip-component=1 -C dust \
  && cd dust \
  && source "$HOME/.cargo/env" \
  && RUSTFLAGS="-Zfmt-debug=none -Zlocation-detail=none -Ctarget-feature=-crt-static" \
    cargo build --config 'profile.release.opt-level = "z"' \
      --config 'profile.release.panic = "abort"' \
      -Z build-std=std,panic_abort \
      -Z build-std-features="optimize_for_size" \
      --target ${build_platform}-unknown-linux-musl \
      --release \
      --locked \
  && upx -q --best --lzma --overlay=strip target/${build_platform}-unknown-linux-musl/release/dust \
  && cp target/${build_platform}-unknown-linux-musl/release/dust /usr/bin \
  && dust --version

# yazi
RUN wget -q https://github.com/sxyazi/yazi/archive/refs/tags/v${yazi_version}.tar.gz -O yazi.tar.gz \
  && echo "${yazi_checksum} yazi.tar.gz" | sha256sum -c - \
  && mkdir yazi \
  && tar xf yazi.tar.gz --strip-component=1 -C yazi \
  && cd yazi \
  && source "$HOME/.cargo/env" \
  && RUSTFLAGS="-Zfmt-debug=none -Zlocation-detail=none -Ctarget-feature=-crt-static" \
    cargo build --config 'profile.release.opt-level = "z"' \
      -Z build-std=std,panic_abort \
      -Z build-std-features="optimize_for_size" \
      --target ${build_platform}-unknown-linux-musl \
      --release \
      --locked \
  && cp target/${build_platform}-unknown-linux-musl/release/yazi /usr/bin \
  && upx -q --best --lzma --overlay=strip target/${build_platform}-unknown-linux-musl/release/ya \
    target/${build_platform}-unknown-linux-musl/release/yazi \
  && cp target/${build_platform}-unknown-linux-musl/release/ya /usr/bin \
  && yazi --version

# bat
RUN wget -q https://github.com/sharkdp/bat/archive/refs/tags/v${bat_version}.tar.gz -O bat.tar.gz \
  && echo "${bat_checksum} bat.tar.gz" | sha256sum -c - \
  && mkdir bat \
  && tar xf bat.tar.gz --strip-component=1 -C bat \
  && cd bat \
  && source "$HOME/.cargo/env" \
  && RUSTFLAGS="-Zfmt-debug=none -Zlocation-detail=none -Ctarget-feature=-crt-static" \
    cargo build --config 'profile.release.opt-level = "z"' \
      --config 'profile.release.panic = "abort"' \
      -Z build-std=std,panic_abort \
      -Z build-std-features="optimize_for_size" \
      --target ${build_platform}-unknown-linux-musl \
      --release \
      --locked \
  && upx -q --best --lzma --overlay=strip target/${build_platform}-unknown-linux-musl/release/bat \
  && cp target/${build_platform}-unknown-linux-musl/release/bat /usr/bin \
  && bat --version

# xh
RUN wget -q https://github.com/ducaale/xh/archive/refs/tags/v${xh_version}.tar.gz -O xh.tar.gz \
  && echo "${xh_checksum} xh.tar.gz" | sha256sum -c - \
  && mkdir xh \
  && tar xf xh.tar.gz --strip-component=1 -C xh \
  && cd xh \
  && source "$HOME/.cargo/env" \
  && RUSTFLAGS="-Zfmt-debug=none -Zlocation-detail=none -Ctarget-feature=-crt-static" \
    cargo build --config 'profile.release.lto = true' \
      --config 'profile.release.opt-level = "z"' \
      --config 'profile.release.panic = "abort"' \
      --config 'profile.release.codegen-units = 1' \
      -Z build-std=std,panic_abort \
      -Z build-std-features="optimize_for_size" \
      --target ${build_platform}-unknown-linux-musl \
      --release \
      --locked \
  && upx -q --best --lzma --overlay=strip target/${build_platform}-unknown-linux-musl/release/xh \
  && cp target/${build_platform}-unknown-linux-musl/release/xh /usr/bin \
  && xh --version

# atac
RUN wget -q https://github.com/Julien-cpsn/ATAC/archive/refs/tags/v${atac_version}.tar.gz -O atac.tar.gz \
  && echo "${atac_checksum} atac.tar.gz" | sha256sum -c - \
  && mkdir atac \
  && tar xf atac.tar.gz --strip-component=1 -C atac \
  && cd atac \
  && source "$HOME/.cargo/env" \
  && RUSTFLAGS="-Zfmt-debug=none -Zlocation-detail=none -Ctarget-feature=-crt-static" \
    cargo build --config 'profile.release.lto = true' \
      --config 'profile.release.opt-level = "z"' \
      --config 'profile.release.panic = "abort"' \
      --config 'profile.release.codegen-units = 1' \
      -Z build-std=std,panic_abort \
      -Z build-std-features="optimize_for_size" \
      --target ${build_platform}-unknown-linux-musl \
      --release \
      --locked \
  && upx -q --best --lzma --overlay=strip target/${build_platform}-unknown-linux-musl/release/atac \
  && cp target/${build_platform}-unknown-linux-musl/release/atac /usr/bin

# fish
RUN wget -q https://github.com/fish-shell/fish-shell/archive/refs/tags/${fish_version}.tar.gz -O fish.tar.gz \
  && echo "${fish_checksum} fish.tar.gz" | sha256sum -c - \
  && mkdir fish \
  && tar xf fish.tar.gz --strip-component=1 -C fish \
  && cd fish \
  && CXXFLAGS="-flto=auto" \
  && cmake -B build -G Ninja \
    -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_INSTALL_SYSCONFDIR=/etc -DCMAKE_INSTALL_LIBDIR=lib \
    -DCMAKE_BUILD_TYPE=MinSizeRel -DBUILD_DOCS=FALSE -DFISH_USE_SYSTEM_PCRE2=OFF \
    -DWITH_GETTEXT=OFF -DBUILD_SHARED_LIBS=NO \
  && cmake --build build --target all \
  && upx --best --lzma --overlay=strip build/fish \
  && cmake --install build

# zellij
RUN wget -q https://github.com/zellij-org/zellij/archive/refs/tags/v${zellij_version}.tar.gz -O zellij.tar.gz \
  && echo "${zellij_checksum} zellij.tar.gz" | sha256sum -c - \
  && mkdir zellij \
  && tar xf zellij.tar.gz --strip-component=1 -C zellij \
  && cd zellij \
  && source "$HOME/.cargo/env" \
  && RUSTFLAGS="-Ctarget-feature=-crt-static" \
    cargo build --config 'profile.release.opt-level = "z"' \
      --config 'profile.release.panic = "abort"' \
      --config 'profile.release.codegen-units = 1' \
      --target ${build_platform}-unknown-linux-musl \
      --release \
      --locked \
  && upx -q --best --lzma --overlay=strip target/${build_platform}-unknown-linux-musl/release/zellij \
  && cp target/${build_platform}-unknown-linux-musl/release/zellij /usr/bin

FROM alpine:3.20.3 AS stage2
ARG naginata_url
ARG naginata_authors
ARG naginata_vendor
ARG naginata_user
ARG naginata_uid
ARG build_revision
ARG build_version
ARG build_created_at
ARG helix_dir

LABEL org.opencontainers.image.source="${naginata_url}" \
      org.opencontainers.image.vendor="${naginata_vendor}" \
      org.opencontainers.image.revision="${build_revision}" \
      org.opencontainers.image.version="${build_version}" \
      org.opencontainers.image.title="Naginata Core" \
      org.opencontainers.image.description="The core image for Naginata." \
      org.opencontainers.image.url="${naginata_url}" \
      org.opencontainers.image.authors="${naginata_authors}" \
      org.opencontainers.image.created="${build_created_at}"

COPY --from=stage1 /usr/bin/grc /usr/bin/grcat /usr/bin/hx /usr/bin/fzf \
  /usr/bin/btm /usr/bin/zoxide /usr/bin/eza /usr/bin/rg /usr/bin/dust \
  /usr/bin/yazi /usr/bin/ya /usr/bin/bat /usr/bin/xh /usr/bin/atac \
  /usr/bin/fish /usr/bin/zellij /usr/bin/
COPY --from=stage1 ${helix_dir} ${helix_dir}
COPY --from=stage1 /etc/grc.conf /etc/
COPY --from=stage1 /usr/share/grc /usr/share/grc
COPY --from=stage1 /usr/share/fish /usr/share/fish
COPY --from=stage1 /etc/fish /etc/fish
COPY motd /etc/motd
COPY skel /etc/skel

ENV HELIX_DEFAULT_RUNTIME=${helix_dir}/runtime
ENV REQUESTS_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt

# TODO: do some magic and replace grc with a smaller footprint coloriser thingy.
RUN apk update -q \
  && apk add --upgrade --no-cache ca-certificates sudo upx \
    openssh-client-common openssh-client-default libgcc libstdc++ \
  && update-ca-certificates \
  # CVE-2019-5021
  sed -i -e 's/^root::/root:!:/' /etc/shadow \
  && echo "%wheel ALL=(ALL:ALL) NOPASSWD:ALL" > /etc/sudoers.d/${naginata_user} \
  && adduser -D ${naginata_user} -u $naginata_uid -G wheel -s /usr/bin/fish \
    -h /home/${naginata_user}

USER ${naginata_user}:wheel
WORKDIR /home/${naginata_user}

# starship
RUN wget -q https://starship.rs/install.sh -O install-starship.sh \
  && sh install-starship.sh -y 1> /dev/null \
  && sudo upx --best --lzma --overlay=strip /usr/local/bin/starship \
  # cleanup
  && rm install-starship.sh \
  && sudo apk del upx \
  && sudo apk cache clean


COPY local.fish /home/${naginata_user}/.local.fish

CMD ["/usr/bin/fish"]
