---
title: Alamofire加载自签名证书	
date: 2021-5-21 22:38:38
tag: iOS
categories: 开发问题
---

##### 使用Alamofire加载自签名证书遇到的问题

```swift
FAILURE: Error Domain=NSURLErrorDomain Code=-1202 “The certificate for this server is invalid. You might be connecting to a server that is pretending to be “portal” which could put your confidential information at risk.”
```

##### Demo

[httpsTruncated](https://github.com/jackfrow/httpsTruncated)

##### 报错原因

https = http + tls

在tls的过程中,有一步操作是会验证服务端的证书,在iOS中URLSession对于证书的验证默认是不会让自签名的证书通过验证

##### 解决

找到URLSession中默认对证书进行处理的部分,对它进行自定义处理。

URLSession对证书的处理会被回调到delegate的方法中,具体方法是

```swift
urlSession(_ session: URLSession,
                         task: URLSessionTask,
                         didReceive challenge: URLAuthenticationChallenge,
                         completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void)
```

af中对证书的处理部分,这部分的大概步奏是

1.它会根据证书类型来对证书进行处理,

2.取出证书管理者

3.取出具体的证书处理者来完成证书的处理。

4.所以我们的解决思路是自定义证书管理者和具体的证书管理类来进行证书处理。

5.具体的代码部分:[httpsTruncated](https://github.com/jackfrow/httpsTruncated)

```swift
open func urlSession(_ session: URLSession,
                         task: URLSessionTask,
                         didReceive challenge: URLAuthenticationChallenge,
                         completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {
        eventMonitor?.urlSession(session, task: task, didReceive: challenge)

        let evaluation: ChallengeEvaluation
        switch challenge.protectionSpace.authenticationMethod {
        case NSURLAuthenticationMethodServerTrust:
            evaluation = attemptServerTrustAuthentication(with: challenge)
        case NSURLAuthenticationMethodHTTPBasic, NSURLAuthenticationMethodHTTPDigest, NSURLAuthenticationMethodNTLM,
             NSURLAuthenticationMethodNegotiate, NSURLAuthenticationMethodClientCertificate:
            evaluation = attemptCredentialAuthentication(for: challenge, belongingTo: task)
        default:
            evaluation = (.performDefaultHandling, nil, nil)
        }

        if let error = evaluation.error {
            stateProvider?.request(for: task)?.didFailTask(task, earlyWithError: error)
        }

        completionHandler(evaluation.disposition, evaluation.credential)
    }

    /// Evaluates the server trust `URLAuthenticationChallenge` received.
    ///
    /// - Parameter challenge: The `URLAuthenticationChallenge`.
    ///
    /// - Returns:             The `ChallengeEvaluation`.
    func attemptServerTrustAuthentication(with challenge: URLAuthenticationChallenge) -> ChallengeEvaluation {
        let host = challenge.protectionSpace.host

        guard challenge.protectionSpace.authenticationMethod == NSURLAuthenticationMethodServerTrust,
              let trust = challenge.protectionSpace.serverTrust
        else {
            return (.performDefaultHandling, nil, nil)
        }

        do {
            guard let evaluator = try stateProvider?.serverTrustManager?.serverTrustEvaluator(forHost: host) else {
                return (.performDefaultHandling, nil, nil)
            }

            try evaluator.evaluate(trust, forHost: host)

            return (.useCredential, URLCredential(trust: trust), nil)
        } catch {
            return (.cancelAuthenticationChallenge, nil, error.asAFError(or: .serverTrustEvaluationFailed(reason: .customEvaluationFailed(error: error))))
        }
    }
```

