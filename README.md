# Linphone4Android
LinPhone은 VoIP 또는 VoIP (Voice over IP)로, GPL을 따르는 개방형 소스 네트워크 비디오 전화 시스템이며, 그 주요 이유는 다음과 같습니다. . Linphone은 SIP 프로토콜을 사용하며 표준 오픈 소스 VoIP 시스템으로, 무료 SIP 기반 오디오 / 비디오 서버를 포함한 모든 SIP 기반 VoIP 통신 업체에 linphone을 연결할 수 있습니다. LinPhone은 LinPhone을 기반으로 다운로드하고 다시 개발할 수있는 무료 소프트웨어 (또는 오픈 소스 소프트웨어)입니다. LinPhone은 데스크톱 컴퓨터 (Linux, Windows, MacOSX 및 모바일 장치 : Android, iPhone, Blackberry)에 사용할 수 있습니다.

# 인터페이스
이 프로젝트는 2 차 개발을위한 라이브러리로 사용될 수 있으며 비즈니스 요구에 따라 소스 코드를 수정할 수 있습니다.

## 로그인
``` java
// 스레드 로그인 열기
ServiceWaitThread mThread = new ServiceWaitThread();
mThread.start();

private void syncAccount(String username, String password, String domain) {

    LinphonePreferences mPrefs = LinphonePreferences.instance();
    if (mPrefs.isFirstLaunch()) {
        mPrefs.setAutomaticallyAcceptVideoRequests(true);
//            mPrefs.setInitiateVideoCall(true);
        mPrefs.enableVideo(true);
    }
    int nbAccounts = mPrefs.getAccountCount();
    if (nbAccounts > 0) {
        String nbUsername = mPrefs.getAccountUsername(0);
        if (nbUsername != null && !nbUsername.equals(username)) {
            mPrefs.deleteAccount(0);
            saveNewAccount(username, password, domain);
        }
    } else {
        saveNewAccount(username, password, domain);
        mPrefs.firstLaunchSuccessful();
    }
}

private void saveNewAccount(String username, String password, String domain) {
    LinphonePreferences.AccountBuilder builder = new LinphonePreferences.AccountBuilder(LinphoneManager.getLc())
            .setUsername(username)
            .setDomain(domain)
            .setPassword(password)
            .setDisplayName(Const.LINPHONE_NAME)
            .setTransport(LinphoneAddress.TransportType.LinphoneTransportTcp);

    try {
        builder.saveNewAccount();
    } catch (LinphoneCoreException e) {
        Log.e(e);
    }
}

private class ServiceWaitThread extends Thread {
    public void run() {
        while (!LinphoneService.isReady()) {
            try {
                sleep(30);
            } catch (InterruptedException e) {
                throw new RuntimeException("waiting thread sleep() has been interrupted");
            }
        }
        handler.post(new Runnable() {
            @Override
            public void run() {
                syncAccount(Const.LINPHONE_ACCOUNT, Const.LINPHONE_PWD, Const.HOST);
            }
        });
        mThread = null;
    }
}

```

## 콜연결
``` java

private void callOutgoing(String number) {
    try {
        if (!LinphoneManager.getInstance().acceptCallIfIncomingPending()) {
            String to = String.format("sip:%s@%s", number, Const.HOST);
            LinphoneManager.getInstance().newOutgoingCall(to, displayName);

            startActivity(new Intent()
                    .setClass(this, LinphoneActivity.class)
                    .addFlags(Intent.FLAG_ACTIVITY_NEW_TASK));
        }
    } catch (LinphoneCoreException e) {
        LinphoneManager.getInstance().terminateCall();
    }
}

```

**참고 :이 프로젝트는 로그인과 콜연결을 위한 두 가지 인터페이스만 제공하며, 적응하기 위해 소스 코드를 수정해야 할 수도 있으므로 개발자가 이를 확장 할 수 있기를 바랍니다.。**
