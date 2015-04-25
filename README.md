# Sexhr

Sexhr provides a simple API around XHR2 without limiting any of its power. The
goal is to have a tiny promise-enabled library that makes working with XHR2
simpler.

## API

Sexhr only has one exported function.

### Sexhr (request)

This function performs the request and returns a promise. If the browser making
the request lacks promises, [please include Bluebird](https://github.com/petkaantonov/bluebird)
or another library like it.

`Sexhr()` takes a single objcet that describes the request. Its options are as
follows:

- `url` - (string) The URL we're requesting. If this is absent, an error is thrown.
- `method` - (string) The HTTP method we're sending. Defaults to 'get'.
- `data` - (any) The data we pass to `XMLHttpRequest.send`. This can be a string,
a blob, form data, etc.
- `emulate` - (boolean) If true and `method` is not "get" or "post", then the
value passed to `method` will be put into a URL param in the request. For instance
if `{url: '/slappy', method: 'delete', emulate: true}` is given, then the final
request would be `POST /slappy?_method=DELETE`.
- `response_type` - (string) The type of response we want. See the
[MDN responseType docs](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest#xmlhttprequest-responsetype)
for possible values. Sexhr resolves with the XMLHttpRequest.response object, so
`response_type` will affect what the value the returned promise gets.
- `timeout` - (number) Milliseconds to wait before the request times out (is
passed directly to `XMLHttpRequest.timeout`).
- `headers` - (object) This is a hash of headers to set into the XHR object.
- `on*` - (function(\*)) Allows setting callbacks directly onto the XHR object. For
instance, you could pass `Sexhr({url: '/file/123', onprogress: myprogressfn})`
and `myprogressfn()` would be called whenever the XHR's progress event fired.
Note that `on*` settings are disabled for `onload`, `onerror`, and `onabort`
because Sexhr uses these directly.
- `upload.on*` - (function(\*)) Allows setting upload callbacks directly onto the
`XMLHttpRequest.upload` object. Example:
`Sexhr({url: '/files', method: 'post', upload: {onprogress: myuploadprogress}})`
- `override` - (function(XHR)) If passed, Sexhr will pass its `XMLHttpRequest`
object to this function just before it sends it out, allowing you to make any
needed overrides here. If you choose to override `onload`, `onerror`, or
`onabort` then you void Sexhr's non-existent warranty.

Sexhr makes no assumptions about the data you pass or the data that's handed
back, so it doesn't do anything like JSON encoding/decoding. That's all up to
you.

## License

MIT.

