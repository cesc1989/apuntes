# Install / Uninstall aws cli in MacOS

Related: [[Instalando y Desintalando AWS CLI]]

# AWS CLI Version 1

From [this guide](https://docs.aws.amazon.com/cli/v1/userguide/install-macos.html#awscli-install-osx-pip).

> Watch out for the Python version installed. Python 2.7 was deprecated in 2020.

Using pip:
```bash
pip install awscli

pip uninstall awscli
```

# AWS CLI Version 2

From [this guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).

Follow steps listed in tab “Command line - current user”

Create the XML file described in the guide. Then run:
```bash
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
```

to download the package.

Install with this commando:
```bash
installer -pkg AWSCLIV2.pkg \
	-target CurrentUserHomeDirectory \
	-applyChoiceChangesXML file_created.xml
```

Create symlinks:
```bash
sudo ln -s /Users/francisco/aws-cli/aws /usr/local/bin/aws
sudo ln -s /Users/francisco/aws-cli/aws_completer /usr/local/bin/aws_completer
```

Test and should output something similar:
```bash
which aws

aws --version
```

