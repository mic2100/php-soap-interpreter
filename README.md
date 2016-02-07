# PHP SOAP Interpreter

[![Build Status](https://travis-ci.org/meng-tian/php-soap-interpreter.svg?branch=master)](https://travis-ci.org/meng-tian/php-soap-interpreter)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/meng-tian/php-soap-interpreter/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/meng-tian/php-soap-interpreter/?branch=master)
[![codecov.io](https://codecov.io/github/meng-tian/php-soap-interpreter/coverage.svg?branch=master)](https://codecov.io/github/meng-tian/php-soap-interpreter?branch=master)

A PHP library for interpreting `SOAP 1.1` and `SOAP 1.2` envelopes. It can be used in WSDL or non-WSDL mode. The implementation is built on the top of PHP's `SoapClient` class. If you are not familiar with `SoapClient`, it is recommended to read [SoapClient Manul](http://php.net/manual/en/class.soapclient.php) first.

### Requirement 
PHP 5.4 --enablelibxml --enable-soap

### Install
```
composer require meng-tian/php-soap-interpreter
```

### Usage
You need an `Interpreter` instance to generate SOAP request envelopes or translate SOAP response envelopes. The constructor of `Interpreter` is the same as `SoapClient`. The first parameter is `wsdl`, second parameter is an array of `options`. In `options`, you can specify the `soap_version`, `location`, `uri`, `encoding`, `classmap` and so on. More detailed explanations can be found in [SoapClient::SoapClient](http://php.net/manual/en/soapclient.soapclient.php).

###### Generate request in WSDL mode

```php
$interpreter = new Interpreter('http://www.webservicex.net/length.asmx?WSDL');
$request = $interpreter->request(
    'ChangeLengthUnit',
    [['LengthValue'=>'1', 'fromLengthUnit'=>'Inches', 'toLengthUnit'=>'Meters']]
);

print_r($request);
/*
Output:
Array
(
    [Endpoint] => http://www.webservicex.net/length.asmx
    [SoapAction] => http://www.webserviceX.NET/ChangeLengthUnit
    [Version] => 1
    [Envelope] => <?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ns1="http://www.webserviceX.NET/"><SOAP-ENV:Body><ns1:ChangeLengthUnit><ns1:LengthValue>1</ns1:LengthValue><ns1:fromLengthUnit>Inches</ns1:fromLengthUnit><ns1:toLengthUnit>Meters</ns1:toLengthUnit></ns1:ChangeLengthUnit></SOAP-ENV:Body></SOAP-ENV:Envelope>
)
*/
```

###### Generate request in non-WSDL mode

```php
// In non-WSDL mode, location and uri must be provided as they are required by SoapClient.
$interpreter = new Interpreter(null, ['location'=>'http://www.webservicex.net/length.asmx', 'uri'=>'http://www.webserviceX.NET/']);
$request = $interpreter->request(
    'ChangeLengthUnit',
    [
        new SoapParam('1', 'ns1:LengthValue'),
        new SoapParam('Inches', 'ns1:fromLengthUnit'),
        new SoapParam('Meters', 'ns1:toLengthUnit')
    ],
    ['soapaction'=>'http://www.webserviceX.NET/ChangeLengthUnit']
);

print_r($request);
/*
Output:
Array
(
    [Endpoint] => http://www.webservicex.net/length.asmx
    [SoapAction] => http://www.webserviceX.NET/ChangeLengthUnit
    [Version] => 1
    [Envelope] => <?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ns1="http://www.webserviceX.NET/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:SOAP-ENC="http://schemas.xmlsoap.org/soap/encoding/" SOAP-ENV:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"><SOAP-ENV:Body><ns1:ChangeLengthUnit><ns1:LengthValue xsi:type="xsd:string">1</ns1:LengthValue><ns1:fromLengthUnit xsi:type="xsd:string">Inches</ns1:fromLengthUnit><ns1:toLengthUnit xsi:type="xsd:string">Meters</ns1:toLengthUnit></ns1:ChangeLengthUnit></SOAP-ENV:Body></SOAP-ENV:Envelope>
)
*/
```

###### Translate response envelope

```php
$interpreter = new Interpreter('http://www.webservicex.net/length.asmx?WSDL');
$response = <<<EOD
<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
    <soap:Body>
        <ChangeLengthUnitResponse xmlns="http://www.webserviceX.NET/">
            <ChangeLengthUnitResult>0.025400000000000002</ChangeLengthUnitResult>
        </ChangeLengthUnitResponse>
    </soap:Body>
</soap:Envelope>
EOD;
$response = $interpreter->response($response);

print_r($response);

/*
Output:
stdClass Object
(
    [ChangeLengthUnitResult] => 0.0254
)
*/
```

### Credits
Thanks the free SOAP web serivices hosted from http://www.webservicex.net. They are used for testing this library.

### License
This library is released under [MIT](https://github.com/meng-tian/php-soap-interpreter/blob/master/LICENSE.md) license.

