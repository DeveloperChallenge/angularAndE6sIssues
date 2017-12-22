Angularjs and es6 class issue

Question: I have to use a service created in angular-ecmascript service class and it has method responseError. I want to use this (HttpInterceptor) service in "this.$httpProvider.interceptors.push('httpInterceptor');". But there is some issue That $httpProvider interceptor takes only the **responseError** method only. so, in responseError mothed has use **this** of the class. Now, issue is, this **this** is not working as **responseError** method working as callback function in interceptor. 

### Solution

Please view the following **Issue** example. 

**HttpInterceptor** service looks like this 

```
import { Service } from 'angular-ecmascript/module-helpers';

export default class HttpInterceptor extends Service {
    static $name = 'httpInterceptor';
    static $inject = ['$q','notFoundEventService'];
    event; message;

    responseError(rejection){
        console.log("Test this", this)
        switch (rejection.status){
            case 404: //not Found
                this.event = this.notFoundEventService;
                console.log("mahesh =",this.notFoundEventService);
                HttpInterceptor.message = "Not found ";
                break;
        }
    //
    //     //broadcase event
        HttpInterceptor.event.broadcast(rejection);
    //
        return $q.reject(HttpInterceptor.message);
    }


}
```

**App.config file**

```
configure() {
        this.$compileProvider.debugInfoEnabled(false);
        this.$httpProvider.interceptors.push('httpInterceptor'); //here i have using httpInterceptor service.
}
```

Now, interceptor will use responseError method from the httpInterceptor service where **this** to call the service.
Note: if you use this **httpInerceptor** method directly in other service it will work. It doesn't work if you call as callback function.


#### Solved

**httpInterceptor** service

```
import { Service } from 'angular-ecmascript/module-helpers';

export default class HttpInterceptor extends Service {
    static $name = 'httpInterceptor';
    static $inject = ['$q','notFoundEventService'];


    constructor($q,notFoundEventService){
        super($q,notFoundEventService);
        this.notFoundEventService = notFoundEventService;
        this.event=null;
        this.message= null;

        this.responseError=(rejection)=>{
            switch (rejection.status){
                case 404: //not Found
                    this.event = this.notFoundEventService;
                    this.message = "Not found ";
                    break;
            }
            //
            //     //broadcase event
            this.event.broadcast(rejection);
            //
            return this.$q.reject(this.message);
        }
    }
}
```
