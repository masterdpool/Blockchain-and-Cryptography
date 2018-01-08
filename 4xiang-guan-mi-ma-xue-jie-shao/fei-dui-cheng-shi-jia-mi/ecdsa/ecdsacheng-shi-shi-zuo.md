# ECDSA程式實作

因為以下程式涉及比較多大數運算，但Javascript因為沒有內建BIg-integer之處理，但python有內建，所以以下範例以python來展示\(version 2.7.14\)。

> 以下程式為無使用ECDSA相關第三方套件之範例

```python
# coding=utf-8

# The proven prime
Pcurve = 2**256 - 2**32 - 2**9 - 2**8 - \
    2**7 - 2**6 - 2**4 - 1  

# Number of points in the field
N = 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141

# This defines the curve. y^2 = x^3 + Acurve * x + Bcurve
Acurve = 0
Bcurve = 7  

Gx = 55066263022277343669578718895168534326250603453777594175500187360389116729240
Gy = 32670510020758816978083085130507043184471273380659243275938904335757337482424
GPoint = (Gx, Gy)  # This is our generator point. Tillions of dif ones possible

# Individual Transaction/Personal Information
# replace with any private key
privKey = 75263518707598184987916378021939673586055614731957507592904438851787542395619
# replace with a truly random number
RandNum = 28695618543805844332113829720373285210420739438570883203839696518176414791234
# the hash of your message/transaction
HashOfThingToSign = 86032112319101611046176971828093669637772856272773459297323797145286374828050


def modinv(a, n=Pcurve):  # Extended Euclidean Algorithm/'division' in elliptic curves
    lm, hm = 1, 0
    low, high = a % n, n
    while low > 1:
        ratio = high / low
        nm, new = hm - lm * ratio, high - low * ratio
        lm, low, hm, high = nm, new, lm, low
    return lm % n


# Not true addition, invented for EC. It adds Point-P with Point-Q.
def ECadd(xp, yp, xq, yq):
    m = ((yq - yp) * modinv(xq - xp, Pcurve)) % Pcurve
    xr = (m * m - xp - xq) % Pcurve
    yr = (m * (xp - xr) - yp) % Pcurve
    return (xr, yr)


# EC point doubling, invented for EC. It doubles Point-P.
def ECdouble(xp, yp):
    LamNumer = 3 * xp * xp + Acurve
    LamDenom = 2 * yp
    Lam = (LamNumer * modinv(LamDenom, Pcurve)) % Pcurve
    xr = (Lam * Lam - 2 * xp) % Pcurve
    yr = (Lam * (xp - xr) - yp) % Pcurve
    return (xr, yr)


def EccMultiply(xs, ys, Scalar):  # Double & add. EC Multiplication, Not true multiplication
    if Scalar == 0 or Scalar >= N:
        raise Exception("Invalid Scalar/Private Key")
    ScalarBin = str(bin(Scalar))[2:]
    Qx, Qy = xs, ys
    for i in range(1, len(ScalarBin)):  # This is invented EC multiplication.
        Qx, Qy = ECdouble(Qx, Qy)  # print "DUB", Qx; print
        if ScalarBin[i] == "1":
            Qx, Qy = ECadd(Qx, Qy, xs, ys)  # print "ADD", Qx; print
    return (Qx, Qy)


print
print "******* Public Key Generation *********"
xPublicKey, yPublicKey = EccMultiply(Gx, Gy, privKey)
print "the private key (in base 10 format):"
print privKey
print
print "the public key:"
print  "xPublicKey:", xPublicKey
print  "yPublicKey:", yPublicKey
print

print "******* Signature Generation *********"
xRandSignPoint, yRandSignPoint = EccMultiply(Gx, Gy, RandNum)
r = xRandSignPoint % N
print "r =", r
s = ((HashOfThingToSign + r * privKey) * (modinv(RandNum, N))) % N
print "s =", s

print
print "******* Signature Verification *********>>"
w = modinv(s, N)
xu1, yu1 = EccMultiply(Gx, Gy, (HashOfThingToSign * w) % N)
xu2, yu2 = EccMultiply(xPublicKey, yPublicKey, (r * w) % N)
x, y = ECadd(xu1, yu1, xu2, yu2)
print r == x
print
```

之後會顯示如下訊息

![](/assets/85.png)
