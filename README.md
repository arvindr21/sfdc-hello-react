## SFDC Hello React app

### Prerequisites
* Saleforce Account - https://login.salesforce.com/?locale=in
* Saleforce CLI - https://developer.salesforce.com/docs/atlas.en-us.sfdx_setup.meta/sfdx_setup/sfdx_setup_install_cli.htm
* VSCode - https://code.visualstudio.com/download
* VSCode extensions - https://trailhead.salesforce.com/en/content/learn/projects/quick-start-lightning-web-components/set-up-visual-studio-code

### Setup
* Clone this repo
* `cd force-app/react-components/hello-react`
* `npm install`

### Onetime setup
* `cd sfdc-hello-react`
* `sfdx force:auth:web:login -d -a myapp`
* `sfdx force:org:create -s -f config/project-scratch-def.json -a myapp`
* Verify creation `sfdx force:org:list`
* Update `.forceignore` and add `**/react-components` to ignore bundling our source code

### Deploy and test
* `sfdx force:source:push`
* `sfdx force:org:open`
* Navigate to `Setup > Platform tools > Custom Code > Lightning Components` and here you should see the list of components that were deployed.
* Open `App Launcher > Sales`
* `Setup > Edit Page`
* From the Left hand side Lightning Components, navigate to `Custom` section and `Drag and Drop` the required component on to the page. A preview of the app would be shown
* Click on `Save` > `Activate` > `Assign as Org default` > `Save`
* Click on `<- Back` to navigate to the page and see the output

### Development
To create a new react component, follow the below steps

#### React app

* `cd force-app/react-components`
* `create-react-app mycomp`
* `cd mycomp`
* `npm run eject`
* Navigate to `force-app/react-components/hello-react/config/paths.js` and update `appBuild` from `resolveApp('build')` to `resolveApp('../../main/default/staticresources/mycomp')`
* The above step will copy the build assets of the react component into `force-app/main/default/staticresources` folder.
* Create a file `force-app/main/default/staticresources/mycomp.resource-meta.xml` and update it as below
  ```
    <?xml version="1.0" encoding="UTF-8"?>
    <StaticResource xmlns="http://soap.sforce.com/2006/04/metadata">
        <cacheControl>Private</cacheControl>
        <contentType>application/zip</contentType>
    </StaticResource>

  ```

#### Lightning Web Component
* In VSCode - `ctrl+shft+p` or `cmd+shft+p` -> `SFDC: Create Ligtning Component` -> `mycomp` -> default location (`force-app/main/default/aura/`)
* Update `force-app/main/default/aura/mycomp/mycomp.cmp` as shown below
  ```
  <aura:component implements="flexipage:availableForAllPageTypes" access="global">
    <lightning:card title="LCC Sample React">
      <lightning:container
        aura:id="reactApp"
        src="{!$Resource.mycomp + '/index.html'}"
        onmessage="{!c.handleMessage}"
        onerror="{!c.handleError}"
      />
    </lightning:card>
  </aura:component>
  ```
  In the above snippet update the `src` to point to the `staticresources` and `mycomp` folder
* Optional - Update `force-app/main/default/aura/mycomp/mycomp.css` as shown below
  ```
    .THIS .slds-card__body {
        padding: 0 12px;
    }

    .THIS iframe {
        height: 400px;
    }
  ```
* Finally update `force-app/main/default/aura/myapp/myappController.js` as shown below
  ```
    ({
      handleMessage: function (component, event, helper) {
          var message = event.getParams();
          var navigationEvent = $A.get("e.force:navigateToSObject");
          navigationEvent.setParams({
              "recordId": message.payload.id,
              "slideDevName": "details"
          });
          navigationEvent.fire();
      },

      handleError: function (component, event, helper) {
          var error = event.getParams();
          console.log(error);
      }
    })
  ```
* Now follow the `Deploy` steps from above.