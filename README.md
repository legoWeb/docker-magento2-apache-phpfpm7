# Steps
1. >echo "127.0.0.1 docker.mage2.lcl" | sudo tee -a /etc/hosts

### Install docker
1. >sudo apt-get update

2. >sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common**

3. >curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

4. >sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

5. >sudo apt-get update 

6. >sudo apt-get install docker-ce docker-ce-cli containerd.io
### Check docker
1. >sudo docker run hello-world
### Use without root rights
1. >sudo usermod -aG docker <your-user>
>Remember to log out and back in for this to take effect!
>
### Install docker-compose
1. >sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
### Give rights
2. >sudo chmod +x /usr/local/bin/docker-compose
### Check installed docker-compose
3. >docker-compose --version
### Create and run project -it takes around 20min with first time
4. >docker-compose up -d --build
### Enter to docker-container and continue work into it
5. >docker exec -it bro_app bash
### Install magento 2
6. >composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition .
### Disable module Klarna_Kp because there have mistakes with own Klarna tests (bellow the link to github when describe why need to do it)
7. >bin/magento module:disable Klarna_Kp
[link for github](https://github.com/magento/magento2-functional-testing-framework/issues/329)
### Install magento with composer 
1. >php bin/magento setup:install \
--base-url="http://docker.mage2.lcl/" \
--db-host="127.0.0.1" \
--db-name="mage" \
--db-user="root" \
--db-password="root" \
--admin-firstname="admin" \
--admin-lastname="admin" \
--admin-email="admin@example.com" \
--admin-user="admin" \
--admin-password="admin123" \
--language="en_US" \
--currency="USD" \
--timezone="Europe/Kiev" \
--use-rewrites="1" \
--backend-frontname="admin"
### After change default mode
2. >bin/magento deploy:mode:set developer
### If images don't view
3. >php bin/magento catalog:images:resize
### Work with MFTF 
### Change some configurations in adminhtml side . It is need to stable work mftf tests
1. >bin/magento config:set cms/wysiwyg/enabled disabled
2. >bin/magento config:set admin/security/admin_account_sharing 1
3. >bin/magento config:set admin/security/use_form_key 0
### Require in composer.json
4. >composer require --dev magento/magento2-functional-testing-framework
### Create new .htaccess
5. >cp dev/tests/acceptance/.htaccess.sample dev/tests/acceptance/.htaccess
### Create mftf project 
6. >vendor/bin/mftf build:project
### Correct data under project: bellow the example
7. >nano dev/tests/acceptance/.env
8. >MAGENTO_BASE_URL=http://docker.mage2.lcl/
MAGENTO_BACKEND_NAME=admin
MAGENTO_ADMIN_USERNAME=admin
MAGENTO_ADMIN_PASSWORD=admin123
SELENIUM_HOST=selenium
SELENIUM_PORT=4444
SELENIUM_PROTOCOL=http
SELENIUM_PATH=/wd/hub
MODULE_WHITELIST=Unity_Mftf
CUSTOM_MODULE_PATHS=/var/www/html/app/code/*/*/Test/Mftf
### Generate urn for PHP Storm (in my machine i have error)
9. >vendor/bin/mftf generate:urn-catalog --force .idea
###  Generate and run all tests
10. >vendor/bin/mftf generate:tests
###  Run single test
11. >vendor/bin/mftf run:test AdminLoginTest