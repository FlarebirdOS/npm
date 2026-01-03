pkgname=npm
pkgver=11.7.0
pkgrel=4
pkgdesc="JavaScript package manager"
arch=('x86_64')
url="https://www.npmjs.com"
license=('Artistic-2.0')
depends=(
    'node-gyp'
    'nodejs'
    'nodejs-nopt'
    'semver'
)
makedepends=(
    'git'
    'jq'
)
options=('!zipman')
source=(npm-cli::git+ssh://git@github.com/npm/cli.git#tag=v${pkgver})
sha256sums=(0c8f86be491fd566ebff726d63e886e8a50e8e8665183273e4d3ee7f3d543771)

build() {
    cd npm-cli

    node scripts/resetdeps.js
    node . run build -w docs
}

package() {
    cd npm-cli

    local mod_dir=/usr/lib/node_modules/${pkgname}

    install -vdm755 ${pkgdir}/{usr/{bin,share/bash-completion/completions},${mod_dir}}

    ln -s ${mod_dir}/bin/${pkgname}-cli.js ${pkgdir}/usr/bin/${pkgname}
    ln -s ${mod_dir}/bin/npx-cli.js ${pkgdir}/usr/bin/npx

    mapfile -t mod_files < <(node . pack --ignore-scripts --dry-run --json | jq -r .[].files.[].path)
    cp --parents -a ${mod_files[@]} ${pkgdir}/${mod_dir}
    node . completion > ${pkgdir}/usr/share/bash-completion/completions/npm
    echo 'globalconfig=/etc/npmrc' > ${pkgdir}/${mod_dir}/npmrc

    cd ${pkgdir}/${mod_dir}
    # Remove superfluous scripts
    rm -r bin/{node-gyp-bin,np{m,x}{,.{cmd,ps1}}}

    # Experimental dedup
    rm -r node_modules/{node-gyp,nopt,semver}

    cd man
    # Workaround for https://github.com/npm/cli/issues/780
    local name page sec title
    for page in man5/folders.5 man5/install.5 man7/*.7; do
        sec=${page##*.}
        name=$(basename ${page} .${sec})
        title=${name@U}

        sed -Ei "s/^\.TH \"${title}\"/.TH \"NPM-${title}\"/" ${page}
        sed -Ei 's/^(\.TH "NPM-'"${title}"'" "[^"]+") "[^"]+"/\1 ""/' ${page}

        mv ${page} ${page/\///npm-}
    done

    gzip --no-name man?/*

    # Support both `man` and `npm help`
    local dest sec_dir
    for sec_dir in man?; do
        dest=${pkgdir}/usr/share/man/${sec_dir}
        install -d ${dest}
        ln -rs ${sec_dir}/* ${dest}
    done
}
