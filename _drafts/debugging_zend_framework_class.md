

#for example to test Zend\Http\Client

# Copy the Library/ path to a temporary locatoin
cp -av /var/www/stg-smartcampus/hbmsulib/Library /tmp/


# Make a test php file
vim /tmp/test.php





set_include_path(get_include_path() . PATH_SEPARATOR . '/tmp/Library');
require_once 'Zend/Loader/StandardAutoloader.php';
$loader = new Zend\Loader\StandardAutoloader(array('autoregister_zf' => true));
$loader->register();

use HBMSU\General\Logging;
use Zend\Http\Client;
use Zend\Http\Request;


                $client = new Client("https://sso.test44.hbmsu.cloud/v1", array('sslverifypeer' => false));
                $client->setMethod(Request::METHOD_DELETE);
                $response = $client->send();
                if ($response->isSuccess()) {
                    Logging::logDebug('CAS Logout: Success!', ['CAS URL' => $logoutUrl], __FILE__, __FUNCTION__);
                } else {
                    Logging::logDebug('CAS Logout: Failed', ['CAS URL' => $logoutUrl], __FILE__, __FUNCTION__);
                }


# Run:

php /tmp/test.php
