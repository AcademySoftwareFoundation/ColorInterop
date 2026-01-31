## Color Space Encodings for Displays

**ASWF Color Interop Forum Recommendation**

*2026-01-26 v1.0.0*


### Introduction

The last step in color management is often sending images to a display. In addition, many file formats are designed for storage of pixel values intended for a specific display. Therefore it is crucial to have reliable specifications for display color spaces. Unfortunately, interop of color values intended for a display is often compromised due to a number of factors, including:

* Lack of clarity around the definition of a given display color space encoding  
* Confusion over what metadata should be set to indicate a given display color space  
* Misunderstandings over what transfer function should be used

This document therefore tries to provide clarity around display color spaces with the goal of improving interoperability between systems that send, store, or receive and process pixel values in display color spaces. It attempts to define common display color spaces in a clear and unambiguous manner and provides a set of Color Interop ID strings that may be used in file formats.


### Technical Background Information for the Recommendation

#### Image State

Modern color management (e.g. ACES, OpenColorIO) is based on the notion of Image State as defined in ISO standard 22028-1. In summary, this draws a distinction between color space encodings that are scene-referred and those that are output-referred (also known as display-referred). A scene-referred color space encodes the colorimetry of an object in a scene. A display-referred color space encodes the colorimetry of a reproduction of a scene. The latter is often intended for a very different viewing environment.

Because the human visual system is adaptive, its response varies greatly based on such variables as absolute levels of illumination, viewing environment, and other factors. Therefore, successful image reproduction requires more than simply sending colorimetry -- other information is required. The Image State is an attempt to communicate some of this additional essential contextual information.

The Color Interop Forum Recommendation [Color Space Encodings for Texture Assets and CG Rendering](https://github.com/AcademySoftwareFoundation/ColorInterop/blob/main/Recommendations/01_TextureAssetColorSpaces/TextureAssetColorSpaces.md) 
defines commonly used scene-referred color spaces. In some cases, there is both a scene-referred and corresponding display-referred version of a given encoding function.

In a nutshell, the key difference between scene-referred and display-referred color spaces is that the latter have had a Display Rendering Transform (DRT) applied, whereas the former have not. (Other terms for a DRT are "tone-map" or "view transform". An ACES Output Transform is a good example of a DRT.) This process is often lossy and restricts the dynamic range and color gamut to that of a given display. Therefore knowing the image state is important in many workflows.

Because all of the color spaces in this recommendation have already received the DRT processing necessary to make them ready for presentation on a display, they are considered display-referred.

#### Transfer Functions

Display color space encodings are generally designed for use with integer-based numeric representations (for example, an eight-bit number between 0 and 255) that may be sent to display hardware. They typically have some kind of non-linear transfer function associated with them that makes them more perceptually uniform and thus better suited for integer representations.

In video, it is common to use the term Opto-Electronic Transfer Function (OETF) to describe what a camera should do to encode the signal for efficient storage or transmission and the term Electro-Optical Transfer Function (EOTF) to describe what the display does to get back to linear light. 

These are not always inverses of each other. For example, in HD video, ITU-R BT.709 (Rec.709) is a specification for the OETF (i.e., a camera), and ITU-R BT.1886 (Rec.1886) is a specification for an EOTF (i.e., a display). The net difference between the two is described using the term Opto-Optical Transfer Function (OOTF) and the basic explanation for why this is necessary is to account for the difference in how the human visual system responds to the capture conditions vs. the display conditions.

The present recommendation concerns itself only with the display side and not the camera side. Thus, it is only concerned with the EOTF, that is, the response of the display. It is not concerned with the OETF, that is, the response of the camera. This simplifies matters greatly for several reasons: real cameras don't always adhere to standard OETF functions and the encoding is generally followed by a color correction step which makes the overall OOTF unknown.

Thus, for the purposes of this document, the encode (inverse EOTF) and decode (EOTF) steps are always exactly symmetric inverses of each other. What the OETF and OOTF or color correction steps were does not really matter, the only problem is how to present the intended colors on a given display.

One more note on terminology: The non-linear transfer function often takes the form of a power function (as was found in the physical response of CRT displays) and the exponent being used is often described using the term "gamma". A "gamma-corrected" signal is a signal that has had an inverse power function applied to it that will basically counteract the power function in the display.

#### Linear Transfer Functions

Several color space encodings defined below contain linear transfer functions. These are intended to be used as working spaces for operations such as compositing multiple display-referred elements together as part of a graphical user interface or a window within a computer display. Typically they will then be converted to one of the non-linear color space encodings defined herein before being sent to an actual monitor or display. However, it is important to realize that these are display-referred color spaces that have already had the scene-to-display tone-mapping (i.e., the DRT) applied.

The linear color space encodings are not suitable for integer usage, a floating-point representation should be used. Values may extend below 0.0 to represent values outside the gamut of the primaries.

These color spaces may contain a mixture of SDR and HDR elements. The maximum white of an SDR user interface element would be placed at 1.0. If HDR content is present, values may extend above 1.0. The value of 1.0 serves as a normalization anchor when adjusting image values to accommodate varying amounts of HDR headroom (e.g., as described in SMPTE ST 2094-50).

#### Signaling of Display Capabilities

Modern display color space encodings sometimes allow colors that are outside the range of a given display, either in dynamic range or color gamut. For example, Rec.2100-PQ allows luminance values up to 10,000 nits and a color gamut that would require lasers to fully achieve. The present recommendation does not try to capture specific display capabilities for the following reasons and open questions:

* The underlying standards being referenced here often do not specify these details (e.g., Rec.2100 does not specify the nit level or gamut of a given display).  
* Figuring out how to best parameterize display capabilities is complicated and would require a lot more work. For example, should this be a fixed list, with options such as "P3D65x1000n0005" (this example is from ITU-T H Suppl. 19), or should it be something like the AV1 metadata or ST-2086 which allow arbitrary floating-point values? If the latter, how should that be represented as a fixed interop ID string? Should it include the minimum black level in addition to the max white level? Should it include a reference white in addition to maximum white?  
* This would likely result in an overwhelmingly large list of ID strings which might be confusing and difficult to use. And it would need to expand regularly as more capable hardware hits the market.  
* In color management systems such as OCIO, details such as targeting a maximum nit level and color gamut are actually part of the view transform (i.e., the display rendering transform) rather than the display color space. In other words, the conversion from intended display colorimetry to display code values, doesn't typically depend on the nit level involved (that's already been done), and at that point it's just applying a primary matrix and generic transfer function (such as PQ). The interop ID strings are intended to signal the display color space, which may be used with a variety of view transforms.  
* If details such as max nit level are needed, for example to populate AV1 or ST-2086 metadata, color management systems such as OCIO already allow extracting this from the view transform by evaluating it directly.

This is perhaps a topic the Color Interop Forum could discuss in the future, but was deemed out of scope for the present recommendation.

#### Integer Encodings

In computer graphics, it is customary to use unsigned-integer representations for color values. In systems such as OpenColorIO, these are normalized into a nominal 0.0 to 1.0 floating-point representation for processing by dividing by (2^N - 1), for example, 8-bit integer values are divided by 255. 

However, in video it is customary to leave additional headroom and footroom in the signal to enable integer-based processing, such as by spatial filters, which may generate values outside a nominal [0,1] range. For example, when encoding a luma signal using 10-bit integers, 0.0 is placed at 64 and 1.0 is placed at 940. This is often called a "narrow", "legal", or "SMPTE" range signal, in contrast to "full" range for a computer graphics style signal.

The accompanying OpenColorIO transforms are designed so as to expect and produce full-range signals. When a narrow-range signal is converted to full-range it is important to preserve the extended-range information (i.e., below 64 or above 940). In the provided OpenColorIO config, these would become floating-point values below 0.0 or above 1.0. In order to preserve the extended-range information, the transfer functions are extended above 1.0 by simply continuing the function. To preserve values below 0.0, the transfer functions are mirrored around the origin. In other words, if the positive transfer function response is `f(x)`, the negative values are generated using `–f(abs(x))`.

In addition, the OpenColorIO transforms are defined on RGB signals. In video, a luma-chroma representation (often called YCbCr or YUV) is often what is encoded, so this should be converted to RGB (with the appropriate matrix for the given signal) prior to color management.

Although conversion to RGB and legal to full range conversion may be handled within a color management system, it is often handled outside it since the encoded signal may need spatial processing to decompress from a 4:2:2 or other compressed representation.

#### OpenColorIO Reference Implementation

An OpenColorIO config file is provided as part of this recommendation that serves as an open source reference implementation for how to convert among the color spaces. It is hoped that providing a complete mathematical implementation will minimize confusion regarding any details that may be vague or unclear from the written description. 

One benefit of this approach is that it shows exactly how to convert every color space to every other and therefore highlights the relationships between them. However, in the provided config, the conversions do not attempt to do dynamic range or gamut mapping (e.g. between HDR and SDR color spaces or between wide and narrow gamuts). Values outside of a given destination color space are simply clamped.

Please refer to Annex A for more detail.


### Components specified as part of the Color Space Recommendation

The following pieces of information are provided for each color space:

| Component | Description |
| :---- | :---- |
| User-facing Name | This is the full name of the color space. It is recommended as the user-facing text that should be used in software user interfaces. For example, these are the names that users would see in the color space menus of an application that uses OpenColorIO. |
| Interop ID | This is a shorter name that is intended to be used in file formats. These are constructed so that they are also suitable for use in file paths or as arguments to command-line tools or scripts. In the context of OpenColorIO, these would appear as the `interop_id` (OCIO 2.5, or higher) and in the "aliases" list but do not appear in user-facing menus. Please see this document for more detail: [An ID for Color Interop](https://docs.google.com/document/d/1T94lYbis9uCskL_ZEMxGBF2JryLfZnjxlEoNgRHZzBE/edit?usp=sharing). |
| Transfer Function | The transfer function is the non-linearity that is applied to the RGB values relative to a linear representation. Values are described as "linear" if they are proportional to light energy from a display. For non-linear transfer functions, the description provided is for the function to convert the non-linear encoding to a linear encoding (i.e., the EOTF). |
| Primaries | The color primaries are the CIE 1931 xy chromaticity coordinates for red, green, and blue. The linear RGB values are tristimulus values in the specified coordinates. Positive RGB values define the gamut for those primaries. Note that since the linear color spaces are expected to be represented using floating-point numbers, colors outside the gamut of the primaries may be represented using a negative value for one or more of R, G, or B. |
| White Point | The CIE 1931 xy chromaticity coordinates that the primaries are normalized to. This defines the white point of the viewing environment that the observer is adapted to. |
| Image State | The image state of the color space encoding, as defined in ISO 22028-1. |
| Typical Viewing Environment | Guidance about the viewing environment and display peak white point, especially as related to differences between nominal standards and how signals are typically used in practice. |
| CICP values | ITU-T H.273 [Coding-independent code points for video signal type identification](https://www.itu.int/rec/T-REC-H.273) is widely used in file formats to describe display color spaces. H.273 defines a number of "code points" but only two are relevant here: color primaries and transfer function. |
| Basic | While this document recommends that implementers support all of the color spaces documented here, it is recognized that the full list may be overwhelming to some users who may struggle with which option to choose. Therefore, a few color spaces are singled out as being especially essential for novice users. This shorter core list may, for example, be used if an application wants to present a shorter list of menu options when creating new assets. |
| Notes | Historically, there has been confusion around the implementation of several of these color spaces. This document tries to provide some additional notes and background information which implementers may find useful. |
| References | This includes a reference to official standards, where they exist, or other official documents that describe properties of the color space encoding, such as the color primaries. |


### Display Color Space Recommendation

##### Summary Table — Overview of the Recommendation

| User-facing Name | Interop iD | Transfer Function | Primaries | White Point | Image State |
| :---- | :---- | :---- | :---- | :---- | :---- |
| sRGB - Display | `srgb_rec709_display` | sRGB | Rec.709 | D65 | Display-referred |
| Rec.1886 Rec.709 - Display | `g24_rec709_display` | 2.4 power | Rec.709 | D65 | Display-referred |
| Display P3 - Display | `srgb_p3d65_display` | sRGB | DCI-P3 | D65 | Display-referred |
| Display P3 HDR - Display | `srgbe_p3d65_display` | Extended sRGB | DCI-P3 | D65 | Display-referred |
| ST2084-P3-D65 - Display | `pq_p3d65_display` | PQ | DCI-P3 | D65 | Display-referred |
| Rec.2100-PQ - Display | `pq_rec2020_display` | PQ | Rec.2100 | D65 | Display-referred |
| Rec.2100-HLG - Display | `hlg_rec2020_display` | HLG | Rec.2100 | D65 | Display-referred |
| Gamma 2.2 Rec.709 - Display | `g22_rec709_display` | 2.2 power | Rec.709 | D65 | Display-referred |
| AdobeRGB - Display | `g22_adobergb_display` | ~2.2 power | AdobeRGB | D65 | Display-referred |
| Gamma 2.6 P3-D65 - Display | `g26_p3d65_display` | 2.6 power | DCI-P3 | D65 | Display-referred |
| DCDM G2.6-XYZ-D65 - Display | `g26_xyzd65_display` | 2.6 power | XYZ | D65 | Display-referred |
| DCDM ST2084-XYZ-D65 - Display | `pq_xyzd65_display` | PQ | XYZ | D65 | Display-referred |
| Linear Rec.709 - Display-referred | `lin_rec709_display` | Linear | Rec.709 | D65 | Display-referred |
| Linear P3-D65 - Display-referred | `lin_p3d65_display` | Linear | DCI-P3 | D65 | Display-referred |
| Linear Rec.2020 - Display-referred | `lin_rec2020_display` | Linear | Rec.2020 | D65 | Display-referred |

#### Standard Dynamic Range (SDR) Displays

| User-facing Name | sRGB - Display |
| :---- | :---- |
| Interop ID | `srgb_rec709_display` |
| Transfer Function | sRGB (piecewise) |
| Primaries | Rec.709 – R {x: 0.640, y: 0.330}, G {x: 0.300, y: 0.600}, B {x: 0.150, y: 0.060} |
| White Point | D65 – {x: 0.3127, y: 0.3290} |
| Image State | Display-referred |
| Typ. Viewing Env. | The spec defines both a "typical" viewing environment (office-like illumination of 200 lux or more) and a reference/encoding viewing environment (somewhat dimmer at 64 lux). In both cases, the spec is for a normal (mid-gray) background and surround (20% of peak white). The guidance is to author content based on the reference environment. However, given that sRGB is used as the standard color space for the internet, it is viewed in an extremely wide variety of uncontrolled environments.  The 80 nit white luminance mentioned in the IEC specification should partially be considered an artifact of the era when the standard was written, many devices today will have a substantially brighter white point. Though keep in mind that computer displays are often set to a lower luminance than video displays, due to the extensive use of full white in user interfaces. |
| CICP | Color primaries: 1, Transfer function: 13 |
| Basic | Yes |
| Notes | Due to numerous problems with the IEC standard, there have been many misunderstandings about the transfer function formula. Please refer to Annex B of the Color Interop Forum ["Color Space Encodings for Texture Assets and CG Rendering"](https://github.com/AcademySoftwareFoundation/ColorInterop/blob/main/Recommendations/01_TextureAssetColorSpaces/TextureAssetColorSpaces.md#annex-b-the-srgb-transfer-function) document for a detailed explanation. |
| References | IEC 61966-2-1:1999 |

| User-facing Name | Rec.1886 Rec.709 - Display |
| :---- | :---- |
| Interop ID | `g24_rec709_display` |
| Transfer Function | Power function with exponent: 2.4  |
| Primaries | Rec.709 – R {x: 0.640, y: 0.330}, G {x: 0.300, y: 0.600}, B {x: 0.150, y: 0.060} |
| White Point | D65 – {x: 0.3127, y: 0.3290} |
| Image State | Display-referred |
| Typ. Viewing Env. | Dim surround environment with a value of approximately 5 nits. See SMPTE ST 2080-3 and ITU-R BT.2035. SMPTE ST 2080-1 defines the reference white level as 100 nits. Recent surveys have found the default white point of consumer televisions to be more than double that level (though the viewing environment for consumer TV is often not a dim surround either). For better interop with HDR workflows, the SDR white is sometimes set to 203 nits. |
| CICP | Color primaries: 1, Transfer function: 1 <br><br>Note that the transfer function table in H.273 confusingly gives the equations for the camera OETF rather than the display EOTF. (See H.273 Section 8.2 Note 1.) Unfortunately, the transfer function of "1" is interpreted in many different ways in various products. |
| Basic | Yes |
| Notes | The transfer function is specified in ITU-R BT.1886. That document provides parameters to model the actual display luminance based on a non-zero black. However, for the purposes of color management, it is assumed that the black level is constant among devices so that no correction is needed and Lb may be set to zero, making the transfer function a pure power function. <br><br>As described in the earlier pages of this recommendation, ITU-R BT.709 is for a camera OETF and so the transfer function it provides is not relevant for a display. This has been a source of great confusion in the industry. ITU-R BT.1886 is the more relevant transfer function for what most people loosely refer to as "Rec.709" video. |
| References | [ITU-R BT.1886](https://www.itu.int/rec/R-REC-BT.1886) <br>[ITU-R BT.709](https://www.itu.int/rec/R-REC-BT.709) <br>[SMPTE ST 2080-1](https://pub.smpte.org/doc/st2080-1/) <br>[SMPTE ST 2080-3](https://pub.smpte.org/doc/st2080-3/) <br>[ITU-T H.273 (07/2024)](https://www.itu.int/rec/T-REC-H.273) <br>[ITU-T Series H supplement 19](https://www.itu.int/rec/T-REC-H.Sup19/en)|

| User-facing Name | Display P3 - Display |
| :---- | :---- |
| Interop ID | `srgb_p3d65_display` |
| Transfer Function | sRGB (piecewise) |
| Primaries | DCI-P3 – R {x: 0.680, y: 0.320}, G {x: 0.265, y: 0.690}, B {x: 0.150, y: 0.060} |
| White Point | D65 – {x: 0.3127, y: 0.3290} |
| Image State | Display-referred |
| Typ. Viewing Env. | The documentation on the ICC site specifies the following: Viewing surround: 4.1 nits, Image background (proximal field): 16 nits, Ambient illuminance: 64 lux. |
| CICP | Color primaries: 12, Transfer function: 13 |
| Basic | Yes |
| Notes | In macOS, to obtain the specified transfer function on Apple displays, it is important to configure the Display System Settings to one of the reference presets such as HDR Video, Photography, or Internet & Web that are intended for content creation. In the general purpose default preset, the transfer function is closer to a 2.2 power function. |
| References | [ICC website spec](https://www.color.org/chardata/rgb/DisplayP3.xalter) <br>[Apple developer documentation](https://developer.apple.com/documentation/coregraphics/cgcolorspace/displayp3) <br>[SMPTE ST 2113](https://pub.smpte.org/doc/st2113/) (for the primaries) |

| User-facing Name | Gamma 2.2 Rec.709 - Display |
| :---- | :---- |
| Interop ID | `g22_rec709_display` |
| Transfer Function | Power function with exponent: 2.2 |
| Primaries | Rec.709 – R {x: 0.640, y: 0.330}, G {x: 0.300, y: 0.600}, B {x: 0.150, y: 0.060} |
| White Point | D65 – {x: 0.3127, y: 0.3290} |
| Image State | Display-referred |
| Typ. Viewing Env. | No normative definition, expectation is for general computer usage. |
| CICP | Color primaries: 1, Transfer function: 4 |
| Basic | No |
| Notes | Because of the confusion around the IEC sRGB specification, some display modes labelled "sRGB" actually use a pure 2.2 power function rather than the piecewise function. These should not be considered equivalent since there are noticeable differences in shadow reproduction. However, some creators prefer to author for this color space rather than sRGB since the result will be lifted blacks if played back on a true sRGB display, whereas authoring to sRGB will result in crushed blacks if played back on a Gamma 2.2 display (in the belief that lifting is less objectionable than crushing). |
| References | There is no color space encoding specification, but the primaries may be found in: [ITU-R BT.709](https://www.itu.int/rec/R-REC-BT.709) |

| User-facing Name | AdobeRGB - Display |
| :---- | :---- |
| Interop ID | `g22_adobergb_display` |
| Transfer Function | Power function with exponent: 563 / 256 (= 2.19921875) |
| Primaries | AdobeRGB – R {x: 0.640, y: 0.330}, G {x: 0.210, y: 0.710}, B {x: 0.150, y: 0.060} |
| White Point | D65 – {x: 0.3127, y: 0.3290} |
| Image State | Display-referred |
| Typ. Viewing Env. | No normative definition, expectation is for general computer usage. |
| CICP |  |
| Basic | No |
| Notes |  |
| References | https://www.adobe.com/digitalimag/adobergb.html |

| User-facing Name | Gamma 2.6 P3-D65 - Display |
| :---- | :---- |
| Interop ID | `g26_p3d65_display` |
| Transfer Function | Power function with exponent: 2.6 |
| Primaries | DCI-P3 – R {x: 0.680, y: 0.320}, G {x: 0.265, y: 0.690}, B {x: 0.150, y: 0.060} |
| White Point | D65 – {x: 0.3127, y: 0.3290} |
| Image State | Display-referred |
| Typ. Viewing Env. | Cinema (dark surround) |
| CICP |  |
| Basic | No |
| Notes | Cinema projectors in grading suites are often set up to this color space. |
| References | [SMPTE ST 2113](https://pub.smpte.org/doc/st2113/) (for the primaries) <br>[SMPTE RP 431-2](https://pub.smpte.org/doc/rp431-2/) |

| User-facing Name | DCDM G2.6-XYZ-D65 - Display |
| :---- | :---- |
| Interop ID | `g26_xyzd65_display` |
| Transfer Function | Power function with exponent: 2.6 and headroom scaling <br><br>The headroom scaling is a factor of 48 / 52.37, applied in linear space. This gives flexibility to achieve a range of white points at 48 nits on a DCI-calibrated projector. In the reference implementation, an output of 1.0 corresponds to 52.37 nits. |
| Primaries | CIE XYZ 1931 – R {x:1., y:0.}, G {x:0., y:1.}, B {x:0., y:0.} |
| White Point | D65 – {x: 0.3127, y: 0.3290} |
| Image State | Display-referred |
| Typ. Viewing Env. | Cinema (dark surround) with a 48 nit white point, as per SMPTE ST 431-1 |
| CICP | Color primaries: 10, Transfer function: 17 |
| Basic | No |
| Notes | The provided reference implementation maps the neutral axis to a D65 white point. |
| References | [SMPTE ST 428-1](https://pub.smpte.org/doc/st428-1/20190212-pub/) <br>[SMPTE RP 431-2](https://pub.smpte.org/doc/rp431-2/) <br>[SMPTE EG 432-1](https://pub.smpte.org/doc/eg432-1/) <br>[SMPTE ST 431-1](https://pub.smpte.org/doc/st431-1/)|

#### High Dynamic Range (HDR) Displays

| User-facing Name | Display P3 HDR - Display |
| :---- | :---- |
| Interop ID | `srgbe_p3d65_display` |
| Transfer Function | sRGB (piecewise), extended range. <br><br>The goal of the extended range is to allow the SDR white of a software UI to be placed at 1.0 while HDR highlights in images extend above this. |
| Primaries | DCI-P3 – R {x: 0.680, y: 0.320}, G {x: 0.265, y: 0.690}, B {x: 0.150, y: 0.060} |
| White Point | D65 – {x: 0.3127, y: 0.3290} |
| Image State | Display-referred |
| Typ. Viewing Env. |  |
| CICP | Color primaries: 12, Transfer function: 13 |
| Basic | Yes |
| Notes | Please see the SDR version of this space above for more detail. |
| References |  |

| User-facing Name | ST2084-P3-D65 - Display |
| :---- | :---- |
| Interop ID | `pq_p3d65_display` |
| Transfer Function | ST 2084 (PQ) |
| Primaries | DCI-P3 – R {x: 0.680, y: 0.320}, G {x: 0.265, y: 0.690}, B {x: 0.150, y: 0.060} |
| White Point | D65 – {x: 0.3127, y: 0.3290} |
| Image State | Display-referred |
| Typ. Viewing Env. | Dim surround (see BT.2100) |
| CICP | Color primaries: 12, Transfer function: 16 |
| Basic | Yes |
| Notes |  |
| References | [ST 2084, High Dynamic Range Electro-Optical Transfer Function of Mastering Reference Displays](https://pub.smpte.org/doc/st2084/) <br>[SMPTE ST 2113](https://pub.smpte.org/doc/st2113/) (for the primaries) |

| User-facing Name | Rec.2100-PQ - Display |
| :---- | :---- |
| Interop ID | `pq_rec2020_display` |
| Transfer Function | ST 2084 (PQ) |
| Primaries | Rec.2020 – R {x: 0.708, y: 0.292}, G {x: 0.170, y: 0.797}, B {x: 0.131, y: 0.046} |
| White Point | D65 – {x: 0.3127, y: 0.3290} |
| Image State | Display-referred |
| Typ. Viewing Env. | Dim surround (see BT.2100) |
| CICP | Color primaries: 9, Transfer function: 16 |
| Basic | Yes |
| Notes |  |
| References | [ITU-R BT.2100](https://www.itu.int/rec/R-REC-BT.2100) |

| User-facing Name | Rec.2100-HLG - Display |
| :---- | :---- |
| Interop ID | `hlg_rec2020_display` |
| Transfer Function | ITU-R BT.2100 (HLG) EOTF <br><br>Unlike the other color spaces here, the HLG EOTF has channel cross-talk. For authoring content, the peak white point should be set to 1000 nits, which sets the "system gamma" parameter to 1.2. |
| Primaries | Rec.2020 – R {x: 0.708, y: 0.292}, G {x: 0.170, y: 0.797}, B {x: 0.131, y: 0.046} |
| White Point | D65 – {x: 0.3127, y: 0.3290} |
| Image State | Display-referred |
| Typ. Viewing Env. | Dim surround (see BT.2100) |
| CICP | Color primaries: 9, Transfer function: 18 |
| Basic | Yes |
| Notes | Similar to the Rec.709 situation described above, H.273 provides the OETF rather than the EOTF in the table for transfer function 18. For the purposes of playing back a signal on a display, the EOTF should be used, and the inverse EOTF should be used for encoding content. These are available in ITU-R BT.2100. |
| References | [ITU-R BT.2100](https://www.itu.int/rec/R-REC-BT.2100) |

| User-facing Name | DCDM ST2084-XYZ-D65 - Display |
| :---- | :---- |
| Interop ID | `pq_xyzd65_display` |
| Transfer Function | ST 2084 (PQ) |
| Primaries | CIE XYZ 1931 – R {x:1., y:0.}, G {x:0., y:1.}, B {x:0., y:0.} |
| White Point | D65 – {x: 0.3127, y: 0.3290} |
| Image State | Display-referred |
| Typ. Viewing Env. | Cinema (dark surround) |
| CICP | Color primaries: 10, Transfer function: 16 |
| Basic | No |
| Notes |  |
| References | [DCI High Dynamic Range D-Cinema Addendum](https://documents.dcimovies.com/HDR-Addendum/release/1.2.1/) |

#### Linear Display-referred Color Spaces

| User-facing Name | Linear Rec.709 - Display-referred |
| :---- | :---- |
| Interop ID | `lin_rec709_display` |
| Transfer Function | Linear (please refer to the section above regarding linear transfer functions) |
| Primaries | Rec.709 – R {x: 0.640, y: 0.330}, G {x: 0.300, y: 0.600}, B {x: 0.150, y: 0.060} |
| White Point | D65 – {x: 0.3127, y: 0.3290} |
| Image State | Display-referred |
| Typ. Viewing Env. | No normative definition, expectation is for general computer usage. |
| CICP | Color primaries: 1, Transfer function: 8 |
| Basic | No |
| Notes | This color space is typically used when compositing a graphical user-interface for presentation on SDR or HDR displays. |
| References | [ITU-R BT.709](https://www.itu.int/rec/R-REC-BT.709) (for the primaries) |

| User-facing Name | Linear P3-D65 - Display-referred |
| :---- | :---- |
| Interop ID | `lin_p3d65_display` |
| Transfer Function | Linear (please refer to the section above regarding linear transfer functions) |
| Primaries | DCI-P3 – R {x: 0.680, y: 0.320}, G {x: 0.265, y: 0.690}, B {x: 0.150, y: 0.060} |
| White Point | D65 – {x: 0.3127, y: 0.3290} |
| Image State | Display-referred |
| Typ. Viewing Env. | No normative definition, expectation is for general computer usage. |
| CICP | Color primaries: 12, Transfer function: 8 |
| Basic | No |
| Notes | This color space is typically used when compositing a graphical user-interface for presentation on wide-gamut SDR or HDR displays. |
| References | [SMPTE ST 2113](https://pub.smpte.org/doc/st2113/) (for the primaries) |

| User-facing Name | Linear Rec.2020 - Display-referred |
| :---- | :---- |
| Interop ID | `lin_rec2020_display` |
| Transfer Function | Linear (please refer to the section above regarding linear transfer functions) |
| Primaries | Rec.2020 – R {x: 0.708, y: 0.292}, G {x: 0.170, y: 0.797}, B {x: 0.131, y: 0.046} |
| White Point | D65 – {x: 0.3127, y: 0.3290} |
| Image State | Display-referred |
| Typ. Viewing Env. | No normative definition, expectation is for general computer usage. |
| CICP | Color primaries: 9, Transfer function: 8 |
| Basic | No |
| Notes | This color space is typically used when compositing a graphical user-interface for presentation on wide-gamut SDR or HDR displays. |
| References | [ITU-R BT.2020](https://www.itu.int/rec/R-REC-BT.2020) (for the primaries) |


### General References

Color Interop Forum Recommendation ["Color Space Encodings for Texture Assets and CG Rendering"](https://github.com/AcademySoftwareFoundation/ColorInterop/blob/main/Recommendations/01_TextureAssetColorSpaces/TextureAssetColorSpaces.md) 

[ISO 22028-1:2016 Photography and graphic technology — Extended colour encodings for digital image storage, manipulation and interchange, Part 1: Architecture and requirements](https://www.iso.org/standard/68761.html)

[Colour Science Precis for the CGI Artist](https://colour-science.github.io/colour-science-precis/)

Giorgianni and Madden, *Digital Color Management: Encoding Solutions,* Second Edition (Addison-Wesley Longman Publishing Co., Inc., 2008, ISBN: 978-0-470-51244-9)

Charles Poynton, *Digital Video and HD Algorithms and Interfaces,* Second Edition (Burlington, Mass.: Elsevier/Morgan Kaufmann, 2012)

[ACES Technical Documentation](https://docs.acescentral.com/)

[SMPTE ST 2065-1:2012 – Academy Color Encoding Specification (ACES)](https://pub.smpte.org/doc/st2065-1/)

[SMPTE RP 177-1993 – Derivation of Basic Television Color Equations](https://pub.smpte.org/doc/rp177/)

SMPTE ST 2094-50 – Dynamic Metadata for Color Volume Transform - Application #5 (Broadcast) (to be published)

[OpenColorIO](https://opencolorio.org/)

[ITU-T H.273 Coding-independent code points for video
signal type identification](https://www.itu.int/rec/T-REC-H.273)

[W3C CSS Color Module Level 4](https://www.w3.org/TR/css-color-4/)

[Qt QColorspace](https://doc.qt.io/qt-6/qcolorspace.html)



## Annexes

### Annex A: OCIO Reference Config

An OpenColorIO config file is provided which implements the recommended color spaces. The file is `core-display-config.ocio`, available in the same directory as this document on GitHub.

[OCIO Reference Config](./core-display-config.ocio)

It uses CIE-XYZ (1931) as the display-referred reference space, which means the transforms for all the other color spaces convert to that space. The neutral axis is placed at a chromaticity of D65. For HDR displays, 1.0 in the connection space corresponds to an absolute luminance level of 100 nits. (This scaling makes images easier to view during development.)

OCIO displays, views, and some scene-referred color space examples are included along with the display color spaces to provide a better illustration of how they fit within a more complete color managed workflow. 

OCIO's built-in ACES 1.1 view transforms are used as the example display rendering transforms to convert from scene-referred to display-referred values. These are well-known example DRTs, but in practice, other DRTs may be used. For converting from one display-referred color space to another, a "Video (colorimetric)" view transform is included, which is simply a no-op.

OpenColorIO provides a maximum of single-precision (32-bit float) pixel processing. However, the matrix coefficients and other parameters in the config file are calculated at double-precision and so may be used as a reference source for higher precision processing.

The MatrixTransform in OCIO is a 4x4 matrix, converting RGBA to RGBA. The ordering is such that the first four coefficients produce the red output value, the next four produce green, and so forth.

Because OCIO is oriented towards high-performance image processing, there are a number of speed vs. quality tradeoffs that are available. By default, the library uses a fast approximation of an exact power function and that will slightly affect the conversions between some of these color spaces. Hence, implementers verifying numerical results should set the following environment variable to turn off any optimizations:

`export OCIO_OPTIMIZATION_FLAGS=0`

Note that the library reads this environment variable directly and so generally works even when OCIO is embedded in other applications.

More detail about OCIO and config files is available in the [OpenColorIO documentation](https://opencolorio.readthedocs.io/en/latest/index.html).
