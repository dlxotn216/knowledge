# Vue Axios에서 Accesstoken 만료 시 Token Refresh 후 실패한 Request retry 하는 방법

```javascript
axios.interceptors.response.use(response => {
    if (response.data.result !== undefined) {
        if (response.data.result.hasOwnProperty('accessToken')) {
            store.commit('setAuthentication', response.data.result['accessToken'])
        }
        if (response.data.result.hasOwnProperty('refreshToken')) {
            appSession.setRefreshToken(response.data.result['refreshToken']);
        }
    }
    return response;
}, error => {
    let status = error.response ? error.response.status : 500;
    let errorCode = error.response.data.error.errorCode;
    if (status === 401) {
        if (errorCode === 'ACCESS_TOKEN_EXPIRED' || errorCode === 'TOKEN_CANNOT_PARSE') {
            return loginApi.refreshToken(appSession.getUserId(), appSession.getRefreshToken())
                .then((response) => {
                    const accessToken = response.data.result.accessToken;
                    const refreshToken = response.data.result.refreshToken;

                    appSession.setAccessToken(accessToken);
                    appSession.setRefreshToken(refreshToken);
                    return axios.request(error.config);
                })
        }

        return Promise.reject(error);
    }
```

Axios의 error callback 구문에서 조건에 따라 Promise를 반환하면 된다.
