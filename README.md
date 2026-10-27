# xr-srgb-fix

An OpenXR API layer that fixes washed-out colours in Proton D3D11 VR games.

Proton's D3D11 swapchains get sRGB-encoded twice on their way to the runtime, so the
headset image looks grey and flat. The layer intercepts the app's swapchain images and
runs a small compute shader (`decode.comp`) that undoes the extra encode before frames
are submitted. See the comment at the top of `layer.c` for the details.

It is an *implicit* layer, but it does nothing unless `XR_SRGB_FIX` is set, so it can be
turned on per game (e.g. in a Steam launch option: `XR_SRGB_FIX=1 %command%`).
`XR_SRGB_FIX_DISABLE` forces it off.

## Building

```sh
glslangValidator -V decode.comp -o decode.spv
{ echo "static const uint32_t decode_spv[] = {"; \
  od -An -v -tx4 decode.spv | sed 's/\([0-9a-f]\{8\}\)/0x\1,/g'; echo "};"; } > decode.spv.h
cc -O2 -shared -fPIC -fvisibility=hidden -Wall -o libxr_srgb_fix.so layer.c -ldl -lpthread
```

Needs the OpenXR loader and Vulkan headers. Install the `.so` and point an implicit-layer
manifest (`share/openxr/1/api_layers/implicit.d/*.json`) at it with
`"enable_environment": "XR_SRGB_FIX"`.

Packaged for NixOS in
[landonreekstin/nixos-config](https://github.com/landonreekstin/nixos-config)
(`pkgs/xr-srgb-fix`), which builds it and writes the manifest.
