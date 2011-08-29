INSTALLATION
============

To install, just add the app to your settings.py INSTALLED_APPS like:

    INSTALLED_APPS = [
        ...
        'django_bitcoin',
        ...
    ]

USAGE
=====

You can use the `Wallet` class to do different bitcoin-moving applications. Typical example would be a marketplace-style example. Or job freelance site, where escrow is needed.

Note that while you move bitcoins between Wallet-objects, only bitcoin transactions needed are incoming and outgoing transactions. 
Transactions between the system don't generate "real" bitcoin transactions. Every transaction (except incoming transactions) is logged to "WalletTransaction" object to ease accounting.

    from django_bitcoin import Wallet, currency

    class Profile(models.Model):
        wallet = ForeignKey(Wallet)
        outgoing_bitcoin_address = CharField()

    class Escrow(models.Model):
        wallet = ForeignKey(Wallet)
        buyer_happy = BooleanField(default=False)

    buyer=Profile.objects.create()
    seller=Profile.objects.create()

    purchase=Escrow.objects.create()

    AMOUNT_USD="9.99"

    m=currency.Money(AMOUNT_USD, "USD")
    btc_amount=currency.exchange(m, "BTC")

    print "Send "+str(btc_amount)+" BTC to address "+buyer.wallet.receiving_address()

    sleep(5000) # wait for transaction

    if p1.wallet.total_balance()>=btc_amount:
        p1.send_to_wallet(purchase, btc_amount)

        sleep(1000) # wait for product/service delivery

        if purchase.buyer_happy:
            purchase.wallet.send_to_wallet(seller.wallet)
            seller.wallet.send_to_address(seller.outgoing_bitcoin_address, seller.wallet.total_balance())
        else:
            print "WHY U NO HAPPY"
            #return bitcoins to buyer, 50/50 split or something
        

Also there is older Payment -class, which can be used for simpler things (direct payments etc):

    from django_bitcoin import Payment, new_bitcoin_payment, bitcoinprice
    from decimal import Decimal

    bp=new_bitcoin_payment(Decimal("0.32"))
    bp2=new_bitcoin_payment(Decimal("0.99")/bitcoinprice("USD")) # convert from USD

    if bp.is_paid(minconf=5):
        # send 5% of payment to bitcoin address someaddress
        bp.withdraw_proportion(someaddress, Decimal("5.0"))
        if bp.is_paid() and bp2.is_paid():
            # send from both bp and bp2 (95% from both, only single bitcoin transaction)
            Payment.withdraw_all(someaddress2,  {bp, bp2})

NOTE
====

I don't have time to answer to email support requests, sorry guys.
