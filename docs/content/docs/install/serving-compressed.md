---
title: "Serving GZ version of .dom"
type: docs
weight: 10
# bookFlatSection: false
# bookShowToC: true
---

# Serving the compressed version of .dom

If you want to serve `.dom` on it's smallest possible size, you might want to serve it compressed.

For example, in embedded (eg. IoT) projects the device memory is quite limited. Therefore storing the uncompressed version of the library might not be desired. In such cases, you can always serve `.dom` directly on it's compressed form.

This is achieved simply by setting the correct encoding response header while streaming the compressed library to the client:

```
Content-Encoding: gzip
```

{{< hint "info" >}}
According to HTML standards you should always check the `Accept-Encoding` request header, and proceed with sending the compressed version of the library only if supported.

However *all* the browsers in the market natively support the `gzip` encoding, making this a safe assumption.
{{< /hint >}}

---

## Serving the `.br` version

The latest version of the library is compressed using the [brotli](https://github.com/google/brotli) compression instead of GZip. This is a more efficient compression algorithm, giving us space to pack more features.

Similar to `.gz` version, you can serve the brotli-compressed file using the header:

```
Content-Encoding: br
```

{{< hint "warning" >}}
In contrast to GZip that has been a standardized a long time ago, _brotli_ is a fairly new compression standard. And in this case you should typically check if the client supports this encoding via the `Accept-Encoding` header.

Nonetheless, [more than 90%](https://caniuse.com/#feat=brotli) of the browsers in the market is currently supporting this feature, making the assumption that the clients support it by default a quite safe one.
{{</ hint >}}

## ESP32 Example

If you are writing a firmware for an ESP32 chip, you can directly serve the compressed version of the library, following this guide.

First, convert the compressed version of the library into a C array:

```sh
cat dotdom.min.gz | xxd -i >> dot-dom.h
```

Edit `dot-dom.h`, making sure the bytes are correctly wrapped in a static C array:

```cpp
static const uint8_t DOT_DOM_JS[] = {
  // xxd output ...
  0x1f, 0x8b, 0x08, 0x00, 0x00, 0x00, 0x00, 0x00, 0x02, 0x13, 0x5d, 0x52,
  ...
  // ... up to here
};
```

Then, in your HTTP handler just make sure to set the correct headers:

```cpp
static esp_err_t handle_get_dot_dom(httpd_req_t *req)
{
  httpd_resp_set_hdr(req, "Content-Encoding", "gzip");
  httpd_resp_set_type(req, "application/javascript");
  httpd_resp_send(req, DOT_DOM_JS, sizeof(DOT_DOM_JS));
  return ESP_OK;
}
```


---

{{< topicnav "" "create-webapp" >}}
