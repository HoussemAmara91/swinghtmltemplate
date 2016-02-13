![http://swinghtmltemplate.googlecode.com/files/loginform.jpg](http://swinghtmltemplate.googlecode.com/files/loginform.jpg)

## HTML ##
```
<html>
<head>
    <style type="text/css">
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
        }
    </style>
</head>
<body id="rootPanel" layout="border" style="width: 400; height:300; border: empty 12 12 12 12">
<table>
    <tr>
        <td rowspan="3" class="top">
            <img src="/img/login_icon.gif" alt=""/>
        </td>
        <td width="fill">
            <form id="loginForm">
                <p>Login:</p>
                <input id="login" type="text" align="wrap"/>
                <p>Password:</p>
                <input id="password" type="password" align="wrap" />
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
            <div layout="box" style="x-boxlayout-direction: horizontal;border: empty 0 0 0 6">
                <glue type="horizontal"/>
                <input type="button" text="OK" id="ok" icon="/img/accept.png" />
                <strut type="horizontal" style="width: 6"/>
                <input type="button" text="Cancel" id="cancel" icon="/img/cancel.png"/>
            </div>
        </td>
    </tr>
</table>


</body>
</html>
```

## Java ##
```
package ru.swing.html.example;

import org.jdom.JDOMException;
import ru.swing.html.Bind;
import ru.swing.html.Binder;

import javax.swing.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.IOException;


public class LoginForm {

    @Bind("login")
    private JTextField login;

    @Bind("password")
    private JPasswordField password;

    @Bind("ok")
    private JButton okBtn;

    @Bind("cancel")
    private JButton cancelBtn;

    @Bind("rootPanel")
    private JPanel rootPanel;

    @Bind("result")
    private JLabel result;

    public LoginForm() {
        try {
            Binder.bind(this);
        } catch (JDOMException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        init();
    }

    public void init() {
        okBtn.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                result.setText("Logging in user "+login.getText());
            }
        });

        cancelBtn.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                result.setText("Cancel clicked");
            }
        });
    }

    public JPanel getRootPanel() {
        return rootPanel;
    }

    public static void main(String[] args) throws JDOMException, IOException {

        LoginForm loginForm = new LoginForm();

        JFrame f = new JFrame("Test");
        f.setSize(400, 200);


        f.getContentPane().add(loginForm.getRootPanel());
        f.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        f.setVisible(true);

    }

}
```