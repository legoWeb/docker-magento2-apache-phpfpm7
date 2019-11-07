# STEPS
**echo "127.0.0.1 docker.mage2.lcl" | sudo tee -a /etc/hosts**

##Install Docker
**sudo apt-get update**

**sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common**

**curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -**

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
###CHECK DOCKER
**sudo docker run hello-world**
### USE without ROOT rights
**sudo usermod -aG docker <your-user>**
>Remember to log out and back in for this to take effect!
>
## INSTALL DOCKER-COMPOSE
**sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose**
### даем права 
**sudo chmod +x /usr/local/bin/docker-compose**
### check installed docker-compose
**docker-compose --version**
### создаем и запускаем проект -занимает от 10 -20 минут при первом запуске
**docker-compose up -d --build**
### заходим в контейнер и там продолжаем работать, выполняем комманды уже внутри контейнера
**docker exec -it bro_app bash**
## INSTALL MAGENTO 2
**composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition .**
### отключаем этот модуль, так как есть какая -то несовместимость с тестами
**bin/magento module:disable Klarna_Kp**
<https://github.com/magento/magento2-functional-testing-framework/issues/329>
>ссылка на объяснение почему отключать
>
### установка мадженты с помощью composer 
**php bin/magento setup:install \
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
--backend-frontname="admin"**
###after change default mode
bin/magento deploy:mode:set developer
###if images don't view
php bin/magento catalog:images:resize
## WORK WITH MFTF TESTS
### change some configurations in adminhtml side . It is need to stable work mftf tests
**bin/magento config:set cms/wysiwyg/enabled disabled**
**bin/magento config:set admin/security/admin_account_sharing 1**
**bin/magento config:set admin/security/use_form_key 0**
### прописываем в composer.json
**composer require --dev magento/magento2-functional-testing-framework**
### create new .htaccess
**cp dev/tests/acceptance/.htaccess.sample dev/tests/acceptance/.htaccess**
### create mftf project 
**vendor/bin/mftf build:project**
### корректируем данные под свой проект: ниже приведен возможный экземпляр
**nano dev/tests/acceptance/.env**
**MAGENTO_BASE_URL=http://docker.mage2.lcl/
MAGENTO_BACKEND_NAME=admin
MAGENTO_ADMIN_USERNAME=admin
MAGENTO_ADMIN_PASSWORD=admin123
SELENIUM_HOST=selenium
SELENIUM_PORT=4444
SELENIUM_PROTOCOL=http
SELENIUM_PATH=/wd/hub
MODULE_WHITELIST=Unity_Mftf
CUSTOM_MODULE_PATHS=/var/www/html/app/code/*/*/Test/Mftf
### generate urn for PHP Storm //in my machine i have error
**vendor/bin/mftf generate:urn-catalog --force .idea** 
###  Generate and run all tests
**vendor/bin/mftf generate:tests**
###  Run single test
**vendor/bin/mftf run:test AdminLoginTest**

