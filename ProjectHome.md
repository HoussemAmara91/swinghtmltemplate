Swinghtmltemplate lets you declare swing forms with html markup. Also swinghtmltemplate supports binding of components with jsr 295.

[Documentation](http://code.google.com/p/swinghtmltemplate/wiki/Documentation_0_6)

Download [release-0.6](http://swinghtmltemplate.googlecode.com/files/swinghtmltemplate-0.6.tar.bz2)

Check online examples in [builder tool](http://swinghtmltemplate.googlecode.com/svn/jnlp/launch.jnlp)

## News ##
22.06.2011 - 0.6 release with [xhtmlrenderer](http://code.google.com/p/swinghtmltemplate/wiki/Xhtml_usage) integration, [popup menu](http://code.google.com/p/swinghtmltemplate/wiki/Documentation_0_6#popupMenu) support, fixes

11.05.2011 - 0.5 release with i18n, new tags, placeholders, configuration etc

26.04.2011 - 0.4 release with [ui:composition](http://code.google.com/p/swinghtmltemplate/wiki/Composition) tags (as in facelets)

21.04.2011 - 0.3 release with beans binding support and a set of new tags



---

Below is the example of creating login form. Other [examples](http://code.google.com/p/swinghtmltemplate/w/list?q=label:example) and [usages](http://code.google.com/p/swinghtmltemplate/w/list?q=label:usage) are in [wiki](http://code.google.com/p/swinghtmltemplate/w/list)

## Result ##

![http://swinghtmltemplate.googlecode.com/files/loginform.jpg](http://swinghtmltemplate.googlecode.com/files/loginform.jpg)

## HTML ##
```
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:c="http://www.oracle.com/swing">
<head>
    <link type="text/css" rel="stylesheet" href="loginform.css"/>
</head>
<body id="rootPanel" style="display: border;width: 400; height:300; border: empty 12 12 12 12">
<table>
    <tr>
        <td rowspan="3" class="top">
            <img src="/img/login_icon.gif" alt=""/>
        </td>
        <td width="fill">
            <form id="loginForm">
                <p content="html"><![CDATA[<span style="text-decoration:underline;">Login</span>]]></p>
                <input id="login" type="text" value="${account.name}" align="wrap"/>
                <label for="password">Password:</label>
                <input id="password" type="password" value="${account.password}" align="wrap" />
            </form>
        </td>
    </tr>
    <tr>
        <td>
            <p id="result" class="red"></p>
        </td>
    </tr>
    <tr>
        <td>
            <div style="display: box; x-boxlayout-direction: horizontal;border: empty 0 0 0 6">
                <c:glue type="horizontal"/>
                <input type="button" text="OK" id="ok" icon="/img/accept.png" class="button" onclick="onOkClick"/>
                <c:strut type="horizontal" style="width: 6"/>
                <input type="button" text="Cancel" id="cancel" icon="/img/cancel.png" class="button" onclick="onCancelClick"/>
            </div>
        </td>
    </tr>
</table>


</body>
</html>
```
### css ###
```
.red {
    color: red;
}
.top {
    vertical-align: top;
}
#loginForm {
    x-miglayout-column-constraints: [right]related[grow,fill];
}
body {
    border: compound (empty 12 12 12 12) (compound (etched) (empty 12 12 12 12));
    display: border;
    width: 400; 
    height:300;
}

```

## Java ##
```
package ru.swing.html.example;

import org.jdom.JDOMException;
import ru.swing.html.Bind;
import ru.swing.html.Binder;
import ru.swing.html.DomModel;

import javax.swing.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.IOException;



public class LoginForm extends JPanel{


    @Bind("result")
    private JLabel result;

    private DomModel model;

    @ModelElement("account")
    private Account account = new Account();

    public LoginForm() {
        try {
            model = Binder.bind(this, true);
        } catch (JDOMException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }


    public void onCancelClick(ActionEvent e) {
        result.setText("Cancel clicked");
    }

    public void onOkClick() {
        for (JComponent c : model.select(".button")) {
            c.setEnabled(false);
        }
        result.setText("Logging in user "+account.getName());
    }


    public Account getAccount() {
        return account;
    }

    public void setAccount(Account account) {
        Object old = this.account;
        this.account = account;
        firePropertyChange("account", old, account);
    }



    public static void main(String[] args) throws JDOMException, IOException {

        LoginForm loginForm = new LoginForm();
        Account acc = new Account();
        acc.setName("John Doe");
        loginForm.setAccount(acc);

        JFrame f = new JFrame("Test");
        f.setSize(400, 200);


        f.getContentPane().add(loginForm);
        f.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        f.setVisible(true);

    }

}


```

### maven conf ###
```
    <dependencies>
        <dependency>
            <groupId>com.googlecode.swinghtmltemplate</groupId>
            <artifactId>sht-core</artifactId>
            <version>0.6</version>
        </dependency>
    </dependencies>


    <repositories>
        <repository>
            <id>swinghtmltemplate</id>
            <name>Swing html template repo</name>
            <url>http://swinghtmltemplate.googlecode.com/svn/maven-repository</url>
            <releases>
                <enabled>true</enabled>
                <updatePolicy>never</updatePolicy>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>

```