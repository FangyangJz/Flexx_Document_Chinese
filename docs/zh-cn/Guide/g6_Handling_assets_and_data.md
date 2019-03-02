### 素材(assets)和数据(data)处理

### 素材管理
当编写依赖于特定JS或CSS库的代码时，可以通过将该库与需要它的模块相关联，将该库加载到客户机中。然后，当来自该模块的代码在JS中使用时，Flexx将自动(且仅)加载它。Flexx本身使用的这种机制是一些部件，例如用于 Leaflet 映射或 CodeMirror 编辑器。
```
# Associate asset
flx.assets.associate_asset(__name__, 'mydep.js', js_code)

# Sometimes a more lightweight *remote* asset is prefered
flx.assets.associate_asset(__name__, 'http://some.cdn/lib.css')

# Create component (or Widget) that needs the asset at the client
class MyComponent(flx.JsComponent):
    ....
```
也可以提供不自动加载在主应用程序页面上的素材，例如，对于子页面或web workers:
```
# Register asset
asset_url = flx.assets.add_shared_asset('mydep.js', js_code)
```

### 数据管理
数据可以在每个会话中提供，也可以在会话之间共享:
```
# Add session-specific data. You need to call this inside a PyComponent
# and use the link in the JS component that needs the data.
link = my_component.session.add_data('some_name.png', binary_blob)

# Add shared data. This can be called at the module level.
link = flx.assets.add_shared_data('some_name.png', binary_blob)
```
注意，也可以通过 action 激活, 将数据从Python发送到JS(在本例中数据通过websocket发送)。后者也适用于numpy数组。
