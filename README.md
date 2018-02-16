PHP 7.0.27 with OpenSSL FIPS Complaint Build

# Compile PHP 7.0.27: [In 32 Bit Windows 7]

Reference Link: https://wiki.php.net/internals/windows/stepbystepbuild

1. Install Visual Studio 2015 with VC 14 for PHP 7.0.27 Compilation
2. Get the PHP source, there are two alternatives: http://windows.php.net/download/
3. Get the binary tools available from http://windows.php.net/downloads/php-sdk/. The binary tools archives are named php-sdk-binary-tools-YYYYMMDD.zip, get the latest one
4. Get the libraries on which PHP depends from http://windows.php.net/downloads/php-sdk/deps-7.0-vc14-x86.7z
5. Add FIPS related updates in '...phpdev\vc14\x86\php-7.0.27-src\ext\openssl\openssl.c' file 
6. Create the build directory c:\php-sdk
7. Unpack the binary tools archive into this directory, it should contain three sub-directories: bin, script and share
8. Open the command prompt and enter the build directory:
	> cd c:\php-sdk\
9. Run the buildtree batch script which will create the desired directory structure:
	> bin\phpsdk_buildtree.bat phpdev
10. The buildtree script hasn't been updated for newer versions of VC++ so:
    - If compiling for VC14: copy C:\php-sdk\phpdev\vc9 to C:\php-sdk\phpdev\vc14
11. Extract the PHP source code to C:\php-sdk\phpdev\vc14\x86\
12. In the same directory where you extracted the PHP source there is a deps directory. Where you will need to extract the libraries required to build PHP, which you downloaded in the perevious step (the deps-*.7z archive).
13. Open a file 'C:\php-sdk\phpdev\vc14\x86\php-7.0.27-src\ext\openssl\openssl.c' in editor and add following lines to add FIPS related changes:

```
...
...

	PHP_FUNCTION(openssl_fips_init);
	PHP_FUNCTION(openssl_fips_enabled);

...
...

	ZEND_BEGIN_ARG_INFO(arginfo_openssl_fips_init, 0)
	ZEND_END_ARG_INFO()

	ZEND_BEGIN_ARG_INFO(arginfo_openssl_fips_enabled, 0)
	ZEND_END_ARG_INFO()

...
...

	PHP_FE(openssl_fips_init, arginfo_openssl_fips_init)
	PHP_FE(openssl_fips_enabled, arginfo_openssl_fips_enabled)

...
...

/* {{{ proto void openssl_fips_init(void)
   Turns on the FIPS mode in openssl */
PHP_FUNCTION(openssl_fips_init)
{
	int mode = FIPS_mode(), ret = 0;
	if (mode == 0) {
		ret = FIPS_mode_set(1);
		if (ret != 1) {
			unsigned long err = ERR_get_error();
                        const char* err_str = ERR_error_string(err, NULL);
			php_error_docref(NULL TSRMLS_CC, E_ERROR, "unable to initialise openssl fips mode due to %s", err_str);
			RETURN_FALSE;
		}
	} else {
		php_error_docref(NULL TSRMLS_CC, E_WARNING, "already in fips mode");
	}
	return;
}
/* }}} */

/* {{{ proto int openssl_fips_enabled(void)
   Returns the value of FIPS_mode() which would be 1 if its enabled, otherwise it would be 0. */
PHP_FUNCTION(openssl_fips_enabled)
{
	RETURN_LONG(FIPS_mode());
}
/* }}} */

...
...
```

### Note: Add all above changes, then save it.

After above changes you are now ready do execute PHP compilation.

14. Open Developer Command Prompt for VS2015

 a. Set Variables:
> cd ...\php-sdk\
> bin\phpsdk_setvars.bat

 b. Set Configuration:
> cd ...\Compiling\php-sdk\phpdev\vc14\x86\php-7.0.27-src
> buildconf
> configure --enable-cli --disable-all --disable-zts --with-openssl=ext/openssl/fips,shared

 c. Make Configured Build:
> nmake			[to build]

After above your build will be ready in `Release` directory

Cheers !
