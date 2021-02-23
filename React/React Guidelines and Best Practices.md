*Guides are not rules and should not be followed blindly. Use your head and think.*

### Split components in smaller ones to increase reusability
By splitting components into smaller reusable ones, you're also potentially lowering the bundle size.

```jsx
// BAD
export const SocialButton = ({ type, ...rest }) => {
  if (type === 'google') {
    return (
      <Button onClick={() => window.open('googleAuthLink')} {...rest}>
        <GoogleIcon />
        Login with Google
      </Button>
    );
  }
  if (type === 'facebook') {
    return (
      <Button onClick={() => window.open('facebookAuthLink')} {...rest}>
        <FacebookIcon />
        Login with Facebook
      </Button>
    );
  }
  if (type === 'linkedIn') {
    return (
      <Button onClick={() => window.open('linkedAuthLink')} {...rest}>
        <LinkedInIcon />
        Login with LinkedIn
      </Button>
    );
  }
};

<SocialButton type="google" />
```

```jsx
// GOOD
export const SocialButton = ({ href, icon, children, ...rest }) => {
  return (
    <Button onClick={() => window.open(href)} {...rest}>
      {icon && <span css={{ marginRight: '5px' }}>{icon}</span>}
      {children}
    </Button>
  );
};
export const GoogleSocialButton = (props) => {
  return (
    <SocialButton href={googleAuthLink} icon={<GoogleIcon />} {...props}>
      {t('googleSocialLogin')}
    </SocialButton>
  );
};
export const FacebookSocialButton = (props) => {
  return (
    <SocialButton href={FacebookAuthLink} icon={<FacebookIcon />} {...props}>
      {t('facebookSocialLogin')}
    </SocialButton>
  );
};
```