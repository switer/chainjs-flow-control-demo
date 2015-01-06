chainjs-flow-control-demo
=========================

Use chainjs to deal with your business logic as flow.


Compare two business flow codes

#### Using chainjs:

```js
var MAX_RETRY_TIMES = 3
Chain(function (chain) { // step: data validate
        value = value.trim()
        if (this._validInput(value)) {
            // flow-branch goto
            if (!value) return chain.nextTo('empty')
            chain.next(value)
        } else {
            chain.end()
        }
    })
    .then(function (chain, keyword){ // step: network request
        var retryTimes = chain.data('retry') || 0
        this._search(keyword, function (err, data) {
            if (err) {
                chain.data('retry', retryTimes + 1)
                retryTimes < MAX_RETRY_TIMES ? chain.retry() : chain.end()
            } else {
                chain.next(data)
            }
        })
    })
    .then(function (chain, results) { // step: deal with reponse data
        if (!results || !results.length) return chain.next(false)
        this._renderResults(results)
        chain.next(results.length)
    })
    .branch('empty', function (chain) { // branch: when first step check a empty value input
        // only calling .nextTo() method to goto branch step 
        chain.next(false)
    }) 
    .then(function (chain, isShowList) { // step: View UI react
        isShowList ? this._showResultList() : this._hideResultList()
    })
    .context(this)
    .start()
```

#### Traditional way:

```js
var MAX_RETRY_TIMES = 3
var retryTimes = 0
var that = this

value = value.trim()

function _listShow (isShowList) {
    isShowList ? that._showResultList() : that._hideResultList()
}
function _resultsRender (results) {
    if (!results || !results.length) return
    that._renderResults(results)
}
function _searchRequest(callback) {
    that._search(value, function (err, data) {
        if (err) {
            if (retryTimes ++ < MAX_RETRY_TIMES) _searchRequest(callback)
        } else {
            callback(data)
        }
    })
}

if (this._validInput(value)) {
    // flow-branch goto
    if (!value) {
        return _listShow(false)
    }
    _searchRequest(function (data) {
        _resultsRender(data)
        if (data && data.length) _listShow(true)
        else _listShow(false)
    })
}
```
your can't see the business flow clearly