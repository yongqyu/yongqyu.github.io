---
layout: post
comments_id: 201123
---

블로그를 시작한다. 용도는 아직 미정이나, 의도는 알찬 방과 후 삶을 위한 기록용이다.

블로그는 github pages를 통해 생성했고, jekyll을 통해 여러가지 기능을 쓰고있다. theme은 가장 간단한 [riggraz의 no-style-please](https://github.com/riggraz/no-style-please)를 사용했다.  
추가로 댓글 기능과 통계 기능이 필요했는데, 통계 기능은 theme에서 [goat counter](https://www.goatcounter.com)를 지원해 활성화. 댓글은 [aristath](https://aristath.github.io/blog/static-site-comments-using-github-issues-api)의 것을 추가했다.  
파일 이름 및 구조가 시스템의 구조를 대변하고 있으나, 프로세스를 이해하지는 않는다. 추후 블로그 업데이트가 없는한 (블로그와 이해도를) 현 상태로 유지하려 한다.

---

md를 활용한 포스팅이라 포맷은 익숙하나 활용은 매번 구글링을 통했기에 몇가지 활용법을 남긴다.  
* 링크 : \[name\]\(link\)
* 인용 : \> blockqute
* 코드 : \`\`\` code \`\`\`
* 취소선 : \~\~cancelline\~\~
* 이미지 : \!\[name\]\(path\)

그리고 theme에 같이 들어있던 예제도 추후 활용을 위해 아래에 첨부한다.

---

Lorem ipsum[^1] dolor sit amet, consectetur adipiscing elit. Pellentesque vel lacinia neque. Praesent nulla quam, ullamcorper in sollicitudin ac, molestie sed justo. Cras aliquam, sapien id consectetur accumsan, augue magna faucibus ex, ut ultricies turpis tortor vel ante. In at rutrum tellus.

# Sample heading 1
## Sample heading 2
### Sample heading 3
#### Sample heading 4
##### Sample heading 5
###### Sample heading 6

Mauris viverra dictum ultricies. Vestibulum quis ipsum euismod, facilisis metus sed, varius ipsum. Donec scelerisque lacus libero, eu dignissim sem venenatis at. Etiam id nisl ut lorem gravida euismod.

## Lists

Unordered:

- Fusce non velit cursus ligula mattis convallis vel at metus[^2].
- Sed pharetra tellus massa, non elementum eros vulputate non.
- Suspendisse potenti.

Ordered:

1. Quisque arcu felis, laoreet vel accumsan sit amet, fermentum at nunc.
2. Sed massa quam, auctor in eros quis, porttitor tincidunt orci.
3. Nulla convallis id sapien ornare viverra.
4. Nam a est eget ligula pellentesque posuere.

## Blockquote

The following is a blockquote:

> Suspendisse tempus dolor nec risus sodales posuere. Proin dui dui, mollis a consectetur molestie, lobortis vitae tellus.

## Thematic breaks (<hr>)

Mauris viverra dictum ultricies[^3]. Vestibulum quis ipsum euismod, facilisis metus sed, varius ipsum. Donec scelerisque lacus libero, eu dignissim sem venenatis at. Etiam id nisl ut lorem gravida euismod. **You can put some text inside the horizontal rule like so.**

---
{: data-content="hr with text"}

Mauris viverra dictum ultricies. Vestibulum quis ipsum euismod, facilisis metus sed, varius ipsum. Donec scelerisque lacus libero, eu dignissim sem venenatis at. Etiam id nisl ut lorem gravida euismod. **Or you can just have an clean horizontal rule.**

---

Mauris viverra dictum ultricies. Vestibulum quis ipsum euismod, facilisis metus sed, varius ipsum. Donec scelerisque lacus libero, eu dignissim sem venenatis at. Etiam id nisl ut lorem gravida euismod. Or you can just have an clean horizontal rule.

## Code

Now some code:

```
const ultimateTruth = 'this theme is the best!';
console.log(ultimateTruth);
```

And here is some `inline code`!

## Tables

Now a table:

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

## Images

![theme logo](https://raw.githubusercontent.com/riggraz/no-style-please/master/logo.png){:.ioda}

Logo of *no style, please!* theme[^4]

---

## Table of contents
- [Table of contents](#table-of-contents)
- [The start](#the-start)
- [The middle](#the-middle)
- [The end](#the-end)

Mauris viverra dictum ultricies. Vestibulum quis ipsum euismod, facilisis metus sed, varius ipsum. Donec scelerisque lacus libero, eu dignissim sem venenatis at. Nunc a egestas tortor, sed feugiat leo. Vestibulum porta tincidunt tellus, vitae ornare tortor. Pellentesque habitant morbi tristique senectus et netus et malesuada fames ac turpis egestas. Sed nunc neque, tempor in iaculis non, faucibus et metus. Etiam id nisl ut lorem gravida euismod.

## [The start](#the-start)

Fusce non velit cursus ligula mattis convallis vel at metus. Sed pharetra tellus massa, non elementum eros vulputate non. Suspendisse potenti. Quisque arcu felis, laoreet vel accumsan sit amet, fermentum at nunc. Sed massa quam, auctor in eros quis, porttitor tincidunt orci. Nulla convallis id sapien ornare viverra. Cras nec est lacinia ligula porta tincidunt. Nam a est eget ligula pellentesque posuere. Maecenas quis enim ac risus accumsan scelerisque. Aliquam vitae libero sapien. Etiam convallis, metus nec suscipit condimentum, quam massa congue velit, sit amet sollicitudin nisi tortor a lectus. Cras a arcu enim. Suspendisse hendrerit euismod est ac gravida. Donec vitae elit tristique, suscipit eros at, aliquam augue. In ac faucibus dui. Sed tempor lacus tristique elit sagittis, vitae tempor massa convallis.

## [The middle](#the-middle)

Proin quis velit et eros auctor laoreet. Aenean eget nibh odio. Suspendisse mollis enim pretium, fermentum urna vitae, egestas purus. Donec convallis tincidunt purus, scelerisque fermentum eros sagittis vel. Aliquam ac aliquet risus, tempus iaculis est. Fusce molestie mauris non interdum hendrerit. Curabitur ullamcorper, eros vitae interdum volutpat, lacus magna lacinia turpis, at accumsan dui tortor vel lectus. Aenean risus massa, semper non lectus rutrum, facilisis imperdiet mi. Praesent sed quam quis purus auctor ornare et sed augue. Vestibulum non quam quis ligula luctus placerat sed sit amet erat. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia curae; Fusce auctor, sem eu volutpat dignissim, turpis nibh malesuada arcu, in consequat elit mauris quis sem. Nam tristique sit amet enim vel accumsan. Sed id nibh commodo, dictum sem id, semper quam.

## The end

Donec ex lectus, tempus non lacinia quis, pretium non ipsum. Praesent est nunc, rutrum vel tellus eu, tristique laoreet purus. In rutrum orci sit amet ex ornare, sit amet finibus lacus laoreet. Etiam ac facilisis purus, eget porttitor odio. Suspendisse tempus dolor nec risus sodales posuere. Proin dui dui, mollis a consectetur molestie, lobortis vitae tellus. Vivamus at purus sed urna sollicitudin mattis. Mauris lacinia libero in lobortis pulvinar. Nullam sit amet condimentum justo. Donec orci justo, pharetra ut dolor non, interdum finibus orci. Proin vitae ante a dui sodales commodo ac id elit. Nunc vel accumsan nunc, sit amet congue nunc. Aliquam in lacinia velit. Integer lobortis luctus eros, in fermentum metus aliquet a. Class aptent taciti sociosqu ad litora torquent per conubia nostra, per inceptos himenaeos.

---
{: data-content="footnotes"}

[^1]: this is a footnote. It should highlight if you click on the corresponding superscript number.
[^2]: hey there, i'm using no style please!
[^3]: this is another footnote.
[^4]: this is a very very long footnote to test if a very very long footnote brings some problems or not. I strongly hope that there are no problems but you know sometimes problems arise from nowhere.
