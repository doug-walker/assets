## Color Space Tests -- UsdPreviewSurface

These files test color management in OpenUSD's UsdPreviewSurface.

### Assigning a Color Space to a UsdUVTexture

UsdPreviewSurface uses UsdUVTexture for loading textures from image files. There are multiple potential methods of assigning a color space to a UsdUVTexture:

1.  Using the `sourceColorSpace` input attribute.

2.  Setting `colorSpace` as metadata on the `file` input attribute.

3.  Adding a `colorSpace:name` attribute on the `file` input attribute.

4.  Adding a `colorSpace:name` attribute to a scope enclosing the UsdUVTexture.

[usduvtexture_color_test.usda](./usduvtexture_color_test.usda)

This file tests all four of these possibilities. In addition, it tests what happens when `sourceColorSpace` exists along with one of the other options. The tests are arranged into columns where `sourceColorSpace` is set as follows:

1.  Set to "raw".

2.  Set to "sRGB".

3.  Set to "auto".

4.  The attribute is omitted entirely (this should default to "auto").

5.  Set to "lin_ap1_scene". This is not a legal option for this attribute, but since it is a legal value as a `colorSpace`, one could imagine someone trying to set such a value. The current behavior of OpenUSD is that it seems to ignore the error and treat it as if it were set to "auto", so the textures are set up to look correct in that scenario.

The rows are set up as follows:

1.  Only use `sourceColorSpace` and use an .hdr file for the texture.

2.  Only use `sourceColorSpace` and use a .png file for the texture.

3.  Set `colorSpace` as metadata on the `file` input attribute, with a value "g22_ap1_scene".

4.  Set a `colorSpace:name` attribute on the `file` input attribute, with value "srgb_p3d65_scene".

5.  Set a `colorSpace:name` attribute on the enclosing scope, with value "lin_adobergb_scene".

The test is intended to be rendered in the color space "Linear Rec.709 (sRGB)", which has the color interop ID "lin_rec709_scene". The materials use UsdPreviewSurface `emissiveColor`, so no lights are necessary to render it. In fact, any default lights added by the renderer must first be turned off. There is a default camera set up.

Given that the test intentionally creates situations where conficting color space information is set, it is necessary to state an opinion on what the "correct" behavior is. The opinion taken by this test is that `UsdColorSpaceAPI` values (i.e., `colorSpace` values) should take precedence over `sourceColorSpace`, and the textures are set up based on that point of view. This opinion is held even for the last row, where the `colorSpace` value is set on the enclosing scope.

If the test passes, all of the squares should be within +/- one 8-bit RGB code value of {200, 100, 50}, if an sRGB transfer function has been applied in order to save the result as an integer image file. If the render is saved in scene-linear space as an OpenEXR file, the aim value for all squares is {0.577580, 0.127438, 0.0318960}.

The following image was generated in USD version 26.05 using `usdrecord` and a similar image is possible in `usdview`. Note that one must use the `–disableCameraLight` option to obtain the intended result.

![screenshot](screenshots/usduvtexture_color_test.png)

For this test, Storm is not giving the correct values except for most of the squares in the first two rows. Here are the issues:

- In the first row, the "sRGB" value for `sourceColorSpace` is ignored on .hdr and .exr files (this may be by design, since it would typically not be appropriate for use on those file types). 

- For the third row, in Hydra, `colorSpace` metadata on the file input of a UsdUVTexture seems to be masked by the presence of the `sourceColorSpace` input. Perhaps the fourth column could have worked, but it seems the default of "auto" still takes precedence.

- For the fourth row, Hydra does seem to make a `file:colorSpace:name` value available to renderers, but it is not apparently used by Storm.

- In the fifth row, there does not seem to be a way to obtain the `colorSpace` from the enclosing scope through Hydra, so Hydra-based renderers would not be able to get the fifth row correct.


### License Information

Copyright 2026 Autodesk

CC-BY-ND 4.0 https://creativecommons.org/licenses/by-nd/4.0/
