### Promise封装Axios请求

``` js
import axios from 'axios'

function get(_url,_params){
    return new Promise((resolve,reject)=>{
        axios.get(_url,{_params})
        .then(res=>{
            if(res.status === 200){
                resolve(res.data)
            }
        }).catch(error=>{
            reject(error)
        })
        
    })
}

function post(){
    
}

export default get
```

