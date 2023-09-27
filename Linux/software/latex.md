# Latex

## Installing

Following the instuctions from [tug](https://www.tug.org/texlive/quickinstall.html).

```bash
# change this to be appropriate
export BASE="/usr/local/share/perl5"

export PATH="$BASE/bin${PATH:+:${PATH}}"; export PATH
export PERL5LIB="$BASE/lib/perl5${PERL5LIB:+:${PERL5LIB}}"; export PERL5LIB
export PERL_LOCAL_LIB_ROOT="$BASE${PERL_LOCAL_LIB_ROOT:+:${PERL_LOCAL_LIB_ROOT}}"; export PERL_LOCAL_LIB_ROOT
export PERL_MB_OPT="--install_base \"$BASE\""; export PERL_MB_OPT
export PERL_MM_OPT="INSTALL_BASE=$BASE"
export PERL_MM_OPT
```

```bash
cd /tmp
wget https://mirror.ctan.org/systems/texlive/tlnet/install-tl-unx.tar.gz
zcat < install-tl-unx.tar.gz | tar xf -
cd install-tl-*
sudo perl ./install-tl --no-interaction
```

Update your path.

```bash
export YEAR=2023
export PATH=$PATH:/usr/local/texlive/$YEAR/bin/x86_64-linux
```

