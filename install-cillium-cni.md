**install cillium CLI**
CLI can be used to install Cilium, inspect the state of a Cilium installation, and enable/disable various features

> curl -L --remote-name-all https://github.com/cilium/cilium-cli/releases/latest/download/cilium-linux-amd64.tar.gz{,.sha256sum}
> sha256sum --check cilium-linux-amd64.tar.gz.sha256sum
> sudo tar xzvfC cilium-linux-amd64.tar.gz /usr/local/bin
> rm cilium-linux-amd64.tar.gz{,.sha256sum}

