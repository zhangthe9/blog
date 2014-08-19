---
layout: post
title: nexus pro crack
description:
- nexus pro crack
categories:
- nexus
tags:
---
2.2.1
license-bundle  .jar

编译依赖
http://enunciate.codehaus.org/ 
enunciate-core jar

org.sonatype.licensing.product.internal.DefaultProductLicenseManager
新增方法
<pre class="prettyprint">
public static LicenseKey getLic() {
		LicenseKey localLicenseKey = new NexusLicenseKey(null);
		localLicenseKey.setContactCompany("dd");
		localLicenseKey.setContactCountry("中国");
		localLicenseKey.setContactEmailAddress("admin@ddatsh.com");
		localLicenseKey.setContactName("dd");
		localLicenseKey.setContactTelephone("135xxxxxxxx");
		localLicenseKey.setEffectiveDate(new Date());

		SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
		Date d = null;
		try {
			d = sdf.parse("2999-12-31");
		} catch (ParseException e) {
			e.printStackTrace();
		}
		localLicenseKey.setExpirationDate(d);
		FeatureSet fs = new FeatureSet();
		fs.addFeature(new NexusProfessionalFeature());
		localLicenseKey.setFeatureSet(fs);
		return localLicenseKey;
	}
</pre>
=================
DefaultTrialLicenseManager
<pre class="prettyprint">
public LicenseKey verifyLicense(TrialLicenseParam paramTrialLicenseParam)
		throws LicensingException {
	return DefaultProductLicenseManager.getLic();
}
</pre>
central-secure-nexus-plugin-1.0.4.jar
AuthtokenFetcherImpl
<pre class="prettyprint">

package com.sonatype.central.secure.nexus.plugin.internal;

import javax.inject.Inject;
import javax.inject.Named;

import org.sonatype.sisu.goodies.lifecycle.LifecycleSupport;

@Named
public class AuthtokenFetcherImpl extends LifecycleSupport implements
		AuthtokenFetcher {

	@Inject
	public AuthtokenFetcherImpl() {

	}

	public void setCallback(AuthtokenFetcher.Callback callback) {

	}

	private AuthtokenFetcher.Callback getCallback() {
		return null;
	}

	private AuthtokenClient createClient() {
		return null;
	}

	protected void doStart() throws Exception {

	}

	protected void doStop() throws Exception {

	}

	private byte[] getLicenseRawBytes() {

		return null;
	}

	private void fetchAuthtoken(AuthtokenClient client) throws Exception {

	}

	private void validate(AuthtokenClient client, String authtoken)
			throws Exception {
	}
}
</pre>	
nexus.properties修改
\#Nexus section
nexus-work=${bundleBasedir}/../sonatype-trial/nexus
改成
nexus-work=${bundleBasedir}/../sonatype-work/nexus