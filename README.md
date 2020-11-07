# jar-verifier

Example for verifying signature of a jar signed with jarsinger.exe

## Getting Started

1.  check out the repo

2. Update the following code in Verifier.javaa to check subjectDN of your certificate.

```
    if (!cert.getSubjectDN().toString().contains("OpenFin Inc.")) {
```

2. Build the project.

```sh
   mvn clean package
```

jarverfier.exe should be created in the target directory.

3. Run the executable

```sh
   jarverifier.exe signed_jar_file_name_with_full_path
```

jarverifier.exe returns 0, as exit code, if verification is successful,  1 if  not.

## Run jarvierfier.exe as an app asset in OpenFin Runtime

1. sign jarverifier.exe with your code signing certificate

```sh
signtool sign /f CodeSigning.p12 /p password  /d "OpenFin Inc." /t http://timestamp.comodoca.com/authenticode jarverifier.exe
```

2. zip signed jarverifier.exe to jarverifier.zip

3. host jarverifier.zip with a webserver.

4. add app asset to the app manifest as followings:

```json
  "appAssets": [
    {
      "src": "http://localhost:9002/jarverifier.zip",
      "alias": "jarverifier",
      "target": "jarverifier.exe",
      "version": "1.0.0"
    }
  ],
```

5. In the OpenFin app, invoke the app asset with fin.System.launchExternalProcess API as following:

```javascript
    fin.System.launchExternalProcess({
        alias: 'jarverifier',
        arguments: 'full_path_to_jar',
        certificate: {
            subject: 'O=OpenFin INC., L=New York, S=New York, C=US',
            thumbprint: '069e549c4b96a1f377f2e42a3ed8d1ef5f89a9e0'
        },
        listener: (result) => {
            console.log('the exit code', result.exitCode);
            if (result.exitCode === 0) {
                console.log('good jar');
            } else {
                console.log('invalid jar');
            }
        }
    }).then(processIdentity => {
        console.log(processIdentity);
    }).catch(error => {
        console.log(error);
    });
```
Please make necessary change in 'certificate' section to match your certificate