
# Workarounds for issues caused by corporate newtork overriding SSL certificates

Problem is described [here](https://security.stackexchange.com/questions/177759/what-is-the-best-practice-when-corporate-network-is-overwriting-ssl-certificates).

Solutions for differents dev tools below.

## Workaround: vscode failing to launch in wsl/ubuntu
```bash
cat > ~/.wgetrc <<EOF
check_certificate = off
EOF
```

## Workaround: github certificate, unable to clone a public repo
```bash
# update of root CA known to the system
sudo apt-get install apt-transport-https ca-certificates -y
sudo update-ca-certificates

# fetch certificates and put to the list of locally checked certificates
openssl s_client -showcerts -servername github.com -connect github.com:443 </dev/null 2>/dev/null | sed -n -e '/BEGIN\ CERTIFICATE/,/END\ CERTIFICATE/ p' > github.com.pem
cat github.com.pem | sudo tee -a /etc/ssl/certs/ca-certificates.crt
```

## Workaround: for pip install
```bash
pip install <packagename> --trusted-host pypi.python.org --trusted-host files.pythonhosted.org --trusted-host pypi.org
# example 1
pip install labs-sdk-python --trusted-host pypi.python.org --trusted-host files.pythonhosted.org --trusted-host pypi.org
# example 2
ip install pandas --trusted-host pypi.python.org --trusted-host files.pythonhosted.org --trusted-host pypi.org
```

## Workaround: node / nvm
TBD

## Workaround: for golang modules
> Note: not known workaround for golang v1.12 and older as flag `-insecure` works only in v1.13 and newer
```bash
# v1.13 and newer
go mod -insecure download
```